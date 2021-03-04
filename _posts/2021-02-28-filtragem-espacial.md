---
layout: post
title: Filtragem no domínio espacial I
mathjax: true
---


<div class="message">
  Quarto post da série de processamento digital de imagens com Python e OpenCV, hoje vamos experimentar algumas filtragens com convolução.
</div>

Neste experimento vamos realizar filtragens baseadas na convolução discreta, essa aplicação permite modificar certas características de uma dada imagem, dependendo dos valores utilizados na *máscara* que será aplicada. A convolução digital pode ser definida como:

$$
    g(x,y) = \sum_{s=-a}^{a}\sum_{t=-b}^{b}w(s, t)f(x+s, y+t)
$$

Onde *f* representa a imagem a ser filtrada e *w* a máscara aplicada. A máscara, também conhecida como núcleo da convolução ou *kernel*, representa uma *vizinhança* e uma operação predefinida que é aplicada sobre os pixels que estão sobrepostos pela vizinhança, o resultado gera um novo pixel com as mesmas coordenadas que o centro da máscara, a filtragem se completa a medida que o centro da máscara se desloca pelos pixels da imagem de entrada. Aqui neste experimento vamos utilizar máscaras de tamanho **3 x 3**. Como já foi dito, vamos conseguir resultados diferentes dependendo dos valores da máscara.


## Filtros suavizantes lineares

A aplicação de um filtro suavizante, em uma imagem, faz com que haja um borramento na cena, também podem ser utilizados com o objetivo de reduzir ruídos. A resposta dos filtros suavizantes lineares nada mais são do que a média dos pixels que compõem a *vizinhança*, os modelos que iremos usar são os filtros da **média** e **gaussiano**.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/tabelas.png)

## Filtros aguçantes

Os filtros aguçantes têm como  objetivo destacar transições de intensidade, já que são derivativos, sendo assim estes tipos de filtros são indicados quando se é desejado enfatizar bordas ou outras descontinuidades da imagam, como ruídos, ou caso queira reduzir o foco de áreas com baixas variações de intensidade. Os filtros aguçantes mais usados são os de 1<sup>a</sup> ou 2<sup>a</sup> ordem.
### Filtros de 2<sup>a</sup> ordem

Os filtros de 2<sup>a</sup> ordem são filtros isotrópicos, ou seja, aplicar o filtro em uma imagem e depois rotacionar esta imagem, irá surtir o mesmo efeito de rotacionar a imagem primeiro e em seguida aplicar o filtro. O filtro de 2<sup>a</sup> ordem de uma função $$f(x)$$ unidimensional pode ser definido como:

$$
    \frac{\partial^2 f}{\partial x^2} = f(x+1)+f(x-1)-2f(x)
$$

Aqui vamos utilizar os filtros **Laplaciano** e **Highboost**.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/segundaordem.png)

### Filtros de 1<sup>a</sup> ordem

Os filtros de 1<sup>a</sup> ordem de uma função $$f(x)$$ unidimensional podem ser definidos como:

$$
    \frac{\partial f}{\partial x} = f(x+1)-f(x)

$$
<a id="listagem1"></a>
##### Listagem 1. equalizando.py
{% highlight python %}
import cv2 as cv
import numpy as np

histtam = 256
hrange = np.array([0,256])

#Leitura do vídeo
cap = cv.VideoCapture('videos/histograma.mp4')

width = int(cap.get(3))
height = int(cap.get(4))

histw = 184
histh = 92
binw = int(round(histw/histtam))
accummulate = False

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        #Transformando a imagem para tons de cinza
        frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        #Redimensionamento do vídeo
        frame = cv.resize(frame, (int(width/3), int(height/3)), interpolation=cv.INTER_AREA)

        equalizado = cv.equalizeHist(frame)

        #Processo de cálculo do histograma
        histograma = cv.calcHist([frame], [0], None, [histtam], hrange, accumulate=accummulate)
        histogramaeq = cv.calcHist([equalizado],[0], None, [histtam], hrange, accumulate=accummulate)


        cv.normalize(histograma, histograma, alpha=0, beta=histh, norm_type=cv.NORM_MINMAX)
        cv.normalize(histogramaeq, histogramaeq, alpha=0, beta=histh, norm_type=cv.NORM_MINMAX)

        #Desenhando o histograma
        for i in range(1,histtam):
            cv.line(frame, (binw*(i-1), histh - int(np.round(histograma[i-1]))), (binw*(i), histh - int(np.round(histograma[i]))), (0, 0, 0), thickness=2)
            cv.line(equalizado, (binw * (i - 1), histh - int(np.round(histogramaeq[i - 1]))),(binw * (i), histh - int(np.round(histogramaeq[i]))), (0, 0, 0), thickness=2)

        # Unindo as imagems
        res = np.vstack((frame, equalizado))

        cv.imshow('Frame', res)
        if cv.waitKey(5) & 0xFF == ord('q'):
            break
    else:
        break

cap.release()
cv.destroyAllWindows()

{% endhighlight %}

## Descrição do programa equalizando.py

{% highlight python %}
histtam = 256
hrange = np.array([0,256])

#Leitura do vídeo
cap = cv.VideoCapture('videos/histograma.mp4')

width = int(cap.get(3))
height = int(cap.get(4))

{% endhighlight %}

Inicialmente determinamos o intervalo de valores que nosso histograma irá possuir, como estamos trabalhando apenas com tons de cinza, temos que as amostras estão no intervalo **[0, 255]**. O comando `VideoCapture()` faz a abertura do arquivo de vídeo que iremos utilzar, caso queira realizar a captura através de uma câmera, basta fazer `cap = cv.VideoCapture(0)`. Após abrir o arquivo consultamos as dimensões do arquivo através método `get()`, ela vai retornar o valor de alguma propriedade do arquivo, a escolha é feita através do parâmetro passado, a largura e altura possuem os índices 3 e 4, respectivamente.

{% highlight python %}

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        #Transformando a imagem para tons de cinza
        frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        #Redimensionamento do vídeo
        frame = cv.resize(frame, (int(width/3), int(height/3)), interpolation=cv.INTER_AREA)
{% endhighlight %}

Caso o arquivo de vídeo tenha sido aberto corretamente, damos início ao processamento. o método `read()` coleta, decodifica e nos retorna o próximo *frame* do vídeo, ele também retorna um valor booleano que diz se foi ou não encontrado um novo *frame*, esta saída quem irá nos dizer a hora de parar o processo, ao receber `false` saberemos que chegamos ao fim do arquivo. Daqui em diante as operações serão repetidas para cada novo *frame* lido.

Originalmente o vídeo possui cores, porém como já foi dito, iremos utilizar apenas tons de cinza neste estudo, para isso utilizamos a função `cvtColor()`, ela recebe o *frame* que queremos modificar e realiza uma transformação baseada na *flag* passada, `cv.COLOR_BGR2GRAY` indica que queremos passar de um sistema RGB para um em tons de cinza. Uma última modificação que vamos realizar é diminuir o tamanho da imagem, este passo é apenas para facilitar a visualição dos resultados, visto que o vídeo escolhido possui um tamanho grande. A função `resize()` vai receber: imagem que deve redimensionar; tamanho desejado de saída;  método de interpolação, sendo este opicional.

{% highlight python %}

    equalizado = cv.equalizeHist(frame)

    #Processo de cálculo do histograma
    histograma = cv.calcHist([frame], [0], None, [histtam], hrange, accumulate=accummulate)
    histogramaeq = cv.calcHist([equalizado],[0], None, [histtam], hrange, accumulate=accummulate)


    cv.normalize(histograma, histograma, alpha=0, beta=histh, norm_type=cv.NORM_MINMAX)
    cv.normalize(histogramaeq, histogramaeq, alpha=0, beta=histh, norm_type=cv.NORM_MINMAX)

{% endhighlight %}

Em seguida realizamos a equalização do histograma do *frame* atual com a função `equalizeHist()`, salvando o resultado em outra variável. Agora que temos a imagem antes e depois de equalizada, vamos calcular o histograma de cada uma delas para analisar os resultados. Para este cálculo vamos usar a função `calcHist()` passando os seguintes parâmetros: imagem que vamos operar; lista de dimensões dos canais utilizados para computar o histograma; uma máscara de operação, neste caso não estamos utilizando uma em específico; array com o tamanho do histograma em cada dimensão; intervalo de valores possíveis no histograma; booleano que indica se o histograma deve ser acumulado ou não. Por fim é feita a normalização dos valores dos histogramas, para que possamos comparar os resultados.

A parte final da <a href="#listagem1">Listagem 1</a> prepara o conteúdo que será exibido na tela, onde desenhamos as linhas dos histrogramas e unimos os vídeos em uma única saída. Ao executar, o programa tem o seguinte resultado:

<iframe src="https://www.youtube.com/embed/ioxsFxRCOI8?vq=hd1080&modestbranding=1&showinfo=0" width="545" height="485" frameborder="0"></iframe>
<em class="descricao">Vídeo 1. Exemplo de saída do programa filtros.py</em>

É possível perceber no vídeo que com a equalização do histograma, alguns detalhes da cena que passavam despercebidos se tornaram bem visíveis. A medida que a cena vai aumentando a iluminação vemos que o histograma não equalizado passa a se concentrar no canto direito, indicado que as tonalidades de cinza de maior valor estão em maioria; notamos também como a equalização do histograma espalha as amostras no gráfico, reduzindo assim as diferenças que eram acentuadas na cena original.


## Detectando movimento

Algo interessante que podemos fazer com o uso de histogramas é a detecção de movimentos de uma cena para outra, esta análise é feita com base na comparação entre os dois histogramas, dependendo da diferença entre eles, podemos dizer se houve ou não uma mudança de cena.

##### Listagem 2. movimento.py
{% highlight python %}

import cv2 as cv
import numpy as np

histtam = 256
hrange = np.array([0,256])

accummulate = False

#Leitura do vídeo
cap = cv.VideoCapture('videos/movimento.mp4')

width = int(cap.get(3))
height = int(cap.get(4))

histw = 184
histh = 92
binw = int(round(histw/histtam))
primeiro = True

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        #Transformando a imagem para tons de cinza
        frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        #Redimensionamento do vídeo
        frame = cv.resize(frame, (int(width/5), int(height/5)), interpolation=cv.INTER_AREA)


        #Caso seja o primeiro frame o programa apenas gera o primeiro histograma
        if primeiro:
            histograma1 = cv.calcHist([frame], [0], None, [histtam], hrange, accumulate=accummulate)
            primeiro = False
        # Do segundo frame em diante devemos comparar os dois últimos histogramas, para isso vamos utilizar o método
        # cv.compareHist(), onde iremos escolher a comparação de correlação, logo quanto mais próximo de 1, mais semelhante
        # Vamos adotar que uma comparação que retorne um valor abaixo de 0.9, representa uma mudança na cena
        else:
            histograma2 = cv.calcHist([frame], [0], None, [histtam], hrange, accumulate=accummulate)
            comp = cv.compareHist(histograma1,histograma2, 0)
            #print(comp)

            if comp < 0.999:
                # Escrevendo a mensagem na imagem
                cv.putText(frame,'MOVEMENT DETECTED', (10, int(height/5)-10), cv.FONT_HERSHEY_COMPLEX, 0.7, (255, 255, 255))

            cv.imshow('Frame', frame)
            histograma1 = histograma2
            if cv.waitKey(30) & 0xFF == ord('q'):
                break
    else:
        break

cap.release()
cv.destroyAllWindows()


{% endhighlight %}

## Descrição do programa movimento.py

O código do segundo experimento segue os mesmos passos iniciais do anterior, onde abrimos um arquivo de vídeo e em seguida seguimos uma série de passos em cada *frame* do arquivo.
{% highlight python %}

if primeiro:
    histograma1 = cv.calcHist([frame], [0], None, [histtam], hrange, accumulate=accummulate)
    primeiro = False

{% endhighlight %}

Neste trecho do código nós verificamos se o *frame* que nos encontramos é o primeiro, neste caso ainda não teremos um segundo histograma para realizar comparação, sendo assim armazenamos o histograma da primeira cena e indicamos que o primeiro quadro já foi lido.

{% highlight python %}
else:
    histograma2 = cv.calcHist([frame], [0], None, [histtam], hrange, accumulate=accummulate)
    comp = cv.compareHist(histograma1,histograma2, 0)
    print(comp)

    if comp < 0.9994:
        # Escrevendo a mensagem na imagem
        cv.putText(frame,'MOVEMENT DETECTED', (10, int(height/5)-10), cv.FONT_HERSHEY_COMPLEX, 0.7, (255, 255, 255))

{% endhighlight %}

Para os demais quadros do vídeo, após o calculo do histograma, é feita a comparação do histograma atual com o anterior, esta comeparação é feita pela função `compareHist()` que recebe como parâmetros: o primeiro histograma; o segundo histograma, precisa ter o mesmo tamanho que o primeiro; <a href="http://man.hubwiz.com/docset/OpenCV.docset/Contents/Resources/Documents/d6/dc7/group__imgproc__hist.html#ga994f53817d621e2e4228fc646342d386" target="_blank">método de comparação de histogramas</a>, aqui estamos utilizando o método de correlação. Na comparação por correlação quanto maior for a semelhança entre os quadros, mais próximo de 1 a resposta será, esta reposta possui até 16 casas de precisão, neste experimento vamos utilizar o valor `0.9994`como limiar, onde valores abaixo disso irão representar uma detecção de movimento na cena. Caso haja movimento, escrevemos a mensagem `MOVEMENT DETECTED` na imagem.

{% highlight python %}

cv.imshow('Frame', frame)
histograma1 = histograma2
if cv.waitKey(30) & 0xFF == ord('q'):
    break
{% endhighlight %}

Por fim, apresentamos o resultado em tela, encerrando o processo caso a tecla "q" seja pressionada, note que ao fim do processo atualizamos o valor de `histograma1`. Executando o programa temos o seguinte resultado:

<iframe src="https://youtu.be/ioxsFxRCOI8?vq=hd1080&modestbranding=1&rel=0&iv_load_policy=3" width="1090" height="970" frameborder="0"></iframe>
<em class="descricao">Vídeo 2. Exemplo de saída do programa movimento.py</em>

No vídeo é possível perceber que em determinados momentos, apesar de haver movimento, o programa não os detecta, isso nos diz que a diferença entre as tonalidades de um quadro para o outro não é significativa, caso nosso objetivo fosse diminuir ou aumentar a precisão de detecção, basta regular o valor de limiar. Com isso vimos uma forma relativamente fácil de encontrar movimento em um vídeo com o uso de histogramas. Vou ficando por aqui, bons estudos e até a próxima!