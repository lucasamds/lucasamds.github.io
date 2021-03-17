---
layout: post
title: Filtragem no domínio espacial II
mathjax: true
---


<div class="message">
  Quinto post da série de processamento digital de imagens com Python e OpenCV, desta vez vamos nos aventurar com o efeito fotográfico tilt-shift.
</div>

Na fotografia, o efeito *tilt-shift* é causado pela mudança da orientação e posição da lente em relação ao plano de projeção, o resultado deste efeito é o desfoque seletivo de regiões da cena, *"aumentando"* a profundidade da imagem. Imagens geradas por este efeito se assemelham com uma versão em miniatura da realidade.

Para tentar simular o efeito do *tilt-shift* vamos fazer combinações entre a imagem original e uma versão borrada sua. Para o borramento da imagem original será utilizado o filtro da média, onde teremos uma máscara de tamanho $$ 9 \times 9 $$. 

Neste primeiro experimento queremos aplicar o efeito em imagens estáticas, no segundo experimento deste post vamos trabalhar com vídeos.

<a id="listagem1"></a>
##### Listagem 1. tiltshift.py
{% highlight python %}

import cv2 as cv
import numpy as np
import sys


img = cv.imread('imagens/tilt3.jpg')
width = len(img[0])//2
height = len(img)//2
img = cv.resize(img, (width, height), interpolation=cv.INTER_AREA)

d = 20
l1 = -height//4
l2 = height//2 - l1
alt = l2 + l1
meio = (l2-l1)//2
media = np.ones([9, 9], np.float32)/81
img1 = np.empty((height, width, 3))
img2 = cv.filter2D(img, -1, media)

d_slider_max = 100
alt_slider_max = height
meio_slider_max = height
tbd = 'd'
tbalt = 'altura'
tbm = 'meio'
title_window = 'tilt-shift'
cv.namedWindow(title_window)


def mudad(x):
    global d
    d = x
    criaPond(l1, l2, d)


def mudaalt(x):
    global meio, l1, l2, alt
    novaalt = x
    l1 += novaalt - meio
    l2 += novaalt - meio
    meio = x
    criaPond(l1, l2, d)


def mudameio(x):
    global meio, l1, l2
    novomeio = x
    l1 -= novomeio-meio
    l2 += novomeio-meio
    meio = x
    criaPond(l1, l2, d)


def criaPond(l1, l2, d):
    global img1
    for i in range(height):
        alpha = pondera(i, l1, l2, d)
        img1[i] = cv.addWeighted(img[i], alpha, img2[i], 1-alpha, 0)
    img1 = np.uint8(img1)
    cv.imshow(title_window, img1)


def pondera(x, l1, l2, d):
    if d == 0:
        d = 1
    return (np.tanh((x+l1)/d) - np.tanh((x-l2)/d))/2


criaPond(l1, l2, d)
cv.createTrackbar(tbd, title_window, d, d_slider_max, mudad)
cv.createTrackbar(tbalt, title_window, alt, alt_slider_max, mudaalt)
cv.createTrackbar(tbm, title_window, meio, alt_slider_max, mudameio)

key = cv.waitKey()
if key == ord('s'):
    cv.imwrite('imagens/resultado_tiltshift.png', img1)
else:
    cv.destroyAllWindows()
    sys.exit()

{% endhighlight %}

## Descrição do programa tiltshift.py

{% highlight python %}
d = 20
l1 = -height//4
l2 = height//2 - l1
alt = l2 + l1
meio = (l2-l1)//2
media = np.ones([9, 9], np.float32)/81
img1 = np.empty((height, width, 3))
img2 = cv.filter2D(img, -1, media)
{% endhighlight %}

Após realizar a leitura e o redimensionamento da imagem, iniciamos por criar as variáveis que serão utilzadas nos cálculos, o filtro da média e as matrizes que vão receber as imagens auxiliares durante o programa. Os valores iniciais de `d`, `l1` e `l2`, foram escolhidos de forma que em primeira instância o foco da imagem se encontra em seu centro. 

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/segundaordem.png)

### Filtros de 1<sup>a</sup> ordem

Ao contrário do caso anterior, os filtros de prmeira ordem não são isotrópicos, aqui estamos interessados na magnitude do gradiente., já que a resposta de um gradiente é dada por um vetor e não um escalar. Os filtros de 1<sup>a</sup> ordem de uma função $$f(x)$$ unidimensional podem ser definidos como:

$$
    \frac{\partial f}{\partial x} = f(x+1)-f(x)
$$

Para as máscaras de 1<sup>a</sup> ordem vamos utilizar os operadores de Sobel, eles representam uma aproximação do vetor gradiente. Temos o *G<sub>vert</sub>* que vai enfatizar bordas no sentido vertical da imagem, já o *G<sub>horiz</sub>* as bordas no sentido horizontal.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/primeiraordem.png)

Agora que já conhecemos os filtros que serão aplicados, vejamos o código em python abaixo.



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

<iframe src="https://www.youtube.com/embed/3W_CuVXHHwg?vq=hd1080&showinfo=0&rel=0&iv_load_policy=3" width="560" height="315" frameborder="0"></iframe>
<em class="descricao">Vídeo 2. Exemplo de saída do programa tiltshiftvideo.py</em>

É possível perceber bem no vídeo a diferença entre os filtros suavizantes e aguçantes. Com isso chegamos ao fim de mais um post, bons estudos e até a próxima!