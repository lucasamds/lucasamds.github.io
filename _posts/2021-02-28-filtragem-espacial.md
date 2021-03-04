---
layout: post
title: Filtragem no domínio espacial I
mathjax: true
---


<div class="message">
  Quarto post da série de processamento digital de imagens com Python e OpenCV, hoje vamos experimentar algumas filtragens simples.
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
key = ord('i')


print(f'Width = {width}\nHeight = {height}\nFPS = {cap.get(5)}\nFormat = {cap.get(8)}')

# Filtros
identity = np.array(([0, 0, 0], [0, 1, 0], [0, 0, 0]), dtype=np.float32)
media = np.array([[0.1111, 0.1111, 0.1111], [0.1111, 0.1111, 0.1111], [0.1111, 0.1111, 0.1111]], dtype=np.float32)
gauss = np.array([[0.0625, 0.125, 0.0625], [0.125, 0.25, 0.125], [0.0625, 0.125, 0.0625]], dtype=np.float32)
horizontal = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]], dtype=np.float32)
vertical = np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]], dtype=np.float32)
laplacian = np.array([[0, -1, 0], [-1, 4, -1], [0, -1, 0]], dtype=np.float32)
boost = np.array([[0, -1, 0], [-1, 5.2, -1], [0, -1, 0]], dtype=np.float32)

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
            frameFiltered = cv.filter2D(frame, -1, mask, anchor=(1,1))
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
# Filtros
identity = np.array(([0, 0, 0], [0, 1, 0], [0, 0, 0]), dtype=np.float32)
media = np.array([[0.1111, 0.1111, 0.1111], [0.1111, 0.1111, 0.1111], [0.1111, 0.1111, 0.1111]], dtype=np.float32)
gauss = np.array([[0.0625, 0.125, 0.0625], [0.125, 0.25, 0.125], [0.0625, 0.125, 0.0625]], dtype=np.float32)
horizontal = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]], dtype=np.float32)
vertical = np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]], dtype=np.float32)
laplacian = np.array([[0, -1, 0], [-1, 4, -1], [0, -1, 0]], dtype=np.float32)
boost = np.array([[0, -1, 0], [-1, 5.2, -1], [0, -1, 0]], dtype=np.float32)

mask = identity.copy()

{% endhighlight %}

Neste treco do código nós criamos os filtros que serão utilizadas no programa, em seguida inicializamos a máscara com o filtro identidade, assim nossa imagem filtrada, inicialmente, será igual a imagem de entrada.

Em seguida teremos os processos que vão ocorrer para cada quadro do vídeo.

{% highlight python %}

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
{% endhighlight %}

Ao receber um novo *frame* do vídeo, o programa realiza as operações iniciais necessárias, como redimensionar e transformar a tonalidade da imagem. Após isto o programa vai fazer a escolha do filtro que deve ser aplicado com base no valor da variável `key`, esta por sua vez é inicializada como `key = ord('i')` nas primeiras linhas da <a href="#listagem1">Listagem 1</a>, indicando que o programa de fato inicia com a máscara identidade aplicada. Ao detectar que um novo valor foi salvo em `key`, o programa muda o valor da máscara para o filtro correspondente a letra que foi digitada. Este processo continua enquanto houverem frames no vídeo ou enquanto a tecla *Esc* (27) não for pressionada.

{% highlight python %}

elif key == ord('z'):
    mask = gauss.copy()
    frameFiltered = cv.filter2D(frame, -1, mask, anchor=(1,1))
    printmask(mask)
    mask = laplacian.copy()
    printmask(mask)

{% endhighlight %}

Quando a letra *z* for pressionada, o programa opera o filtro *Laplaciano* sobre uma imagem com o filtro *Gaussiano* aplicado. A função `filter2D()` vai realizar a aplicação da máscara desejada e retornar o resultado, seus parâmetros são: imagem de entrada; a "profundidade" desejada na imagem de saída, quando recebe o valor `-1` ela aplica a mesma profundidade da entrada na saída; o *kernel*, que seria nossa máscara; o ponto âncora do *kernel*. 

{% highlight python %}
 if absolute:
    frameFiltered = cv.convertScaleAbs(frameFiltered)

cv.imshow('Spacial filter', frameFiltered)
key = cv.waitKey(60)
{% endhighlight %}

Caso a opção de ativar o valor absoluto tenha sido ativada, realizamos a operação via a função `convertScaleAbs()`, ela nos retorna o valor absoluto do imagem passada como parâmetro. Por fim, imprimimos na tela o frame atual filtrado, em seguida o programa espera por um breve momento a ativação de uma tecla antes de passar para o próximo quadro. O programa também imprime no console qual a máscara está sendo aplicada atualmente, facilitando o acompanhamento durante a execução.

<iframe src="https://www.youtube.com/embed/ioxsFxRCOI8?vq=hd1080&modestbranding=1&showinfo=0" width="545" height="485" frameborder="0"></iframe>
<em class="descricao">Vídeo 1. Exemplo de saída do programa filtros.py</em>

É possível perceber bem no vídeo a diferença entre os filtros suavizantes e aguçantes. Com isso chegamos ao fim de mais um post, bons estudos e até a próxima!