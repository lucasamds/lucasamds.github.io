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

Ao contrário do caso anterior, os filtros de prmeira ordem não são isotrópicos, aqui estamos interessados na magnitude do gradiente., já que a resposta de um gradiente é dada por um vetor e não um escalar. Os filtros de 1<sup>a</sup> ordem de uma função $$f(x)$$ unidimensional podem ser definidos como:

$$
    \frac{\partial f}{\partial x} = f(x+1)-f(x)
$$

Para as máscaras de 1<sup>a</sup> ordem vamos utilizar os operadores de Sobel, eles representam uma aproximação do vetor gradiente. Temos o *G<sub>vert</sub>* que vai enfatizar bordas no sentido vertical da imagem, já o *G<sub>horiz</sub>* as bordas no sentido horizontal.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/primeiraordem.png)

Agora que já conhecemos os filtros que serão aplicados, vejamos o código em python abaixo.

<a id="listagem1"></a>
##### Listagem 1. filtros.py
{% highlight python %}
import cv2 as cv
import numpy as np

def printmask(mask):
    for i in mask:
        print(i)
    print('\n\n')

cap = cv.VideoCapture('videos/masks.mp4')
width = int(cap.get(3))
height = int(cap.get(4))

frameFiltered = np.zeros((height, width, 3), dtype=np.float32)
absolute = 1
key = 0


print(f'Width = {width}\nHeight = {height}\nFPS = {cap.get(5)}\nFormat = {cap.get(8)}')

# Filtros
identity = np.array(([0, 0, 0], [0, 1, 0], [0, 0, 0]), dtype=np.float32)
media = np.array([[0.1111, 0.1111, 0.1111], [0.1111, 0.1111, 0.1111], [0.1111, 0.1111, 0.1111]])
gauss = np.array([[0.0625, 0.125, 0.0625], [0.125, 0.25, 0.125], [0.0625, 0.125, 0.0625]])
horizontal = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]])
vertical = np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]])
laplacian = np.array([[0, -1, 0], [-1, 4, -1], [0, -1, 0]])
boost = np.array([[0, -1, 0], [-1, 5.2, -1], [0, -1, 0]])

mask = identity.copy()

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        frame = cv.resize(frame, (int(width/2), int(height/2)), interpolation=cv.INTER_AREA)
        frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        cv.imshow('Original', frame)

        frame = np.float32(frame)

        if key == 27:
            break
        elif key == ord('i'):
            mask = identity.copy()
            print(mask)
        elif key == ord('a'):
            absolute = not absolute
        elif key == ord('m'):
            mask = media.copy()
            printmask(mask)
        elif key == ord('g'):
            mask = gauss.copy()
            printmask(mask)
        elif key == ord('h'):
            mask = horizontal.copy()
            printmask(mask)
        elif key == ord('v'):
            mask = vertical.copy()
            printmask(mask)
        elif key == ord('l'):
            mask = laplacian.copy()
            printmask(mask)
        elif key == ord('b'):
            mask = boost.copy()
            printmask(mask)
        elif key == ord('z'):
            mask = gauss.copy()
            frameFiltered = cv.filter2D(frame, -1, mask)
            printmask(mask)
            mask = laplacian.copy()
            printmask(mask)

        frameFiltered = cv.filter2D(frame, -1, mask, anchor=(1, 1))

        if absolute:
            frameFiltered = cv.convertScaleAbs(frameFiltered)

        cv.imshow('Spacial filter', frameFiltered)
        key = cv.waitKey(60)

    else:
        break

cap.release()
cv.destroyAllWindows()

{% endhighlight %}

## Descrição do programa filtros.py

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