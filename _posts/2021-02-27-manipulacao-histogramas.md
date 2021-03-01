---
layout: post
title: Manipulação de histogramas com Python
---


<div class="message">
  Terceiro post da série de processamento digital de imagens com Python e OpenCV, desta vez vamos trabalhar com histogramas de imagens.
</div>

De forma resumida, o histograma é a representração gráfica de uma determinada distribuição numérica, ele tem como objetivo mostrar a frequência que uma determinada amostra aparece na cena. Nestes experimentos vamos trabalhar com um vídeo, pois temos interesse em estudar a varição de histogramas entre uma cena e outra. Para tornar as coisas um pouco mais simples, faremos a leitura do vídeo considerando apenas seus tons de cinza.

## Equalizando um histograma

Neste primeiro experimento nós queremos realizar a equalização de um histograma e analisar o efeito que este processo causa no vídeo. Equalizar significa que iremos mudar a distribuição dos valores de ocorrência de um dado histograma, diminuindo assim as diferenças acentuadas da cena. Quando este efeito é aplicado no processamento de imagens, geralmente conseguimos visualizar detalhes que eram imperceptíveis na figura, já que temos uma normalização do brilho e aumento do contraste.

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
<div class="iframe-container">
    <iframe src="https://www.youtube.com/embed/KtwqHBMgcro" width="640" height="720" frameborder="0"></iframe>
</div>
<em>Vídeo 1. Exemplo de saída do programa equalizando.py</em>

É possível perceber no vídeo que com a equalização do histograma, alguns detalhes da cena que passavam despercebidos se tornaram bem visíveis. A medida que a cena vai aumentando a iluminação vemos que o histograma não equalizado passa a se concentrar no canto direito, indicado que as tonalidades de cinza de maior valor estão em maioria; notamos também como a equalização do histograma espalha as amostras no gráfico, reduzindo assim as diferenças que eram acentuadas na cena original.


## Detectando movimento

Algo interessante que podemos fazer com o uso de histogramas é a detecção de movimentos de uma cena para outra, esta análise é feita com base na comparação entre os dois histogramas, dependendo da diferença entre eles, podemos dizer se houve ou não uma mudança de cena.


{% highlight python %}
##### Listagem 2. movimento.py

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

Para os demais quadros do vídeo, após o calculo do histograma, é feita a comparação do histograma atual com o anterior, esta comeparação é feita pela função `compareHist()` que recebe como parâmetros: primeiro histograma; segundo histograma, precisa ter o mesmo tamanho que o primeiro;<a href="http://man.hubwiz.com/docset/OpenCV.docset/Contents/Resources/Documents/d6/dc7/group__imgproc__hist.html#ga994f53817d621e2e4228fc646342d386">método de comparação de histogramas</a>, aqui estamos utilizando o método de correlação. Na comparação por correlação quanto maior for a semelhança entre os quadros, mais próximo de 1 a resposta será, esta reposta possui até 16 casas de precisão, neste experimento vamos utilizar o valor `0.9994`como limiar, onde valores abaixo disso irão representar uma detecção de movimento na cena. Caso haja movimento, escrevemos a mensagem `MOVEMENT DETECTED` na imagem.

{% highlight python %}

cv.imshow('Frame', frame)
histograma1 = histograma2
if cv.waitKey(30) & 0xFF == ord('q'):
    break
{% endhighlight %}

Por fim, apresentamos o resultado em tela, encerrando o processo caso a tecla "q" seja pressionada, note que ao fim do processo atualizamos o valor de `histograma1`. Executando o programa temos o seguinte resultado:

<iframe width="560" height="315" src="https://www.youtube.com/embed/IcyYyT7gWrM" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

No vídeo é possível perceber que em determinados momentos, apesar de haver movimento, o programa não detecta, isso nos diz que a diferença entre as tonalidades de um quadro para o outro não é significativa, caso nosso objetivo fosse diminuir ou aumentar a precisão de detecção, basta regular o valor de limiar. Com isso vimos uma forma relativamente fácil de encontrar movimento em um vídeo com o uso de histogramas. Vou ficando por aqui, bons estudos e até a próxima!