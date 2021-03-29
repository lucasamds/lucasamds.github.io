---
layout: post
title: Algoritmo de Canny e a arte com pontilhismo
mathjax: true
---


<div class="message">
  Sétimo post da série de processamento digital de imagens, iremos estudar mais um método de detecção de bordas, também vamos criar imagens com a técnica do pontilhismo.
</div>

## Algoritmo de Canny

Em posts anteriores discutimos um pouco sobre filtros que possuem aplicabilidade na detecção de bordas em uma imagem, hoje vamos falar sobre o algoritmo de Canny, geralmente este método apresenta uma melhor performance que os filtros vistos anteriormente. De forma resumida, o algoritmo de Canny segue os seguintes passos:

1. Suavizar a imagem com um filtro Gaussiano
2. Computar a magnitude e o ângulo do gradiente
3. Aplicar uma supressão não máxima na magnitude do gradiente da imagem
4. Utilizar uma análise de conectividade e limiarização dupla, para detectar e estabelecer relações de bordas.

Este breve entendimento do funcionamento do algoritmo de Canny é suficiente para a aplicação que vamos desenvolver hoje, caso sinta vontade de se aprofundar na teoria por trás do processo, indico fortemente que realize uma pesquisa sobre o assunto. Abaixo um exemplo de filtragem com o algoritmo de Canny.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/saidabordas.png)
*Figura 1. Detecção de bordas com o algoritmo de Canny*

Na limiarização iremos utilizar dois limiares, $$ T_1 $$ e $$ T_2 $$, onde $$ T_1 \gt T_2 $$, como é de se esperar, a quantidade de bordas detectadas vai depender dos valores de limiar, é aconselhável que $$ T_2 = 3 \times T_1 $$ ou $$ T_2 = 2 \times T_1 $$. Abaixo um comparativo entre esses dois valores.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/saidabordas2.png)
*Figura 2. À esquerda, T<sub>2</sub> = 2 x T<sub>1</sub>. À direita, T<sub>2</sub> = 3 x T<sub>1</sub>.*

## A arte com pontilhismo

<a id="listagem1"></a>
##### Listagem 1. cannybordas.py
{% highlight python %}

import numpy as np
import cv2 as cv
import random
import sys

img = cv.imread('imagens/3.jpg', cv.IMREAD_GRAYSCALE)
if img is None:
    sys.exit('Imagem não encontrada')
random.seed()
step = 5
step_max = 50
jitter = 20
jitter_max = 100
radius = 3
radius_max = 100
radius_borda = 2
radius_borda_max=10

#img = cv.resize(img, (len(img[0])//2, len(img)//2), interpolation=cv.INTER_AREA)
#cv.imshow('Original', img)
height = img.shape[0]
width = img.shape[1]
pontos = np.full((height, width), 255, dtype=np.uint8)
xrange = np.array(range(0, height))
yrange = np.array(range(0, width))
cv.namedWindow('Pontos', cv.WINDOW_AUTOSIZE)

th1 = 110 #50 img1  130 img2    110 img3
th2 = 140 #130 img1 130 img2    140 img3
bordas = cv.Canny(img, th1, th2)

def mudarange():
    for i in range(0, len(xrange)):
        xrange[i] = xrange[i]*step+step/2

    for i in range(0, len(yrange)):
        yrange[i] = yrange[i]*step+step/2
    desenha()


def desenha():
    random.shuffle(xrange)
    for i in range(0, xrange.shape[0]):
        random.shuffle(yrange)
        for j in range(0, yrange.shape[0]):
            x = i + random.randint(0, (2 * jitter)) - jitter + 1
            if x >= height:
                x = height-1
            y = j + random.randint(0, (2 * jitter)) - jitter + 1
            if y >= width:
                y = width-1
            gray = int(img[x, y])
            color = (gray, gray, gray)
            cv.circle(pontos, (y, x), radius, color, -1, cv.LINE_AA)
    fazbordas()


def fazbordas():
    for i in range(0, height):
        for j in range(0, width):
            if bordas[i, j] == 255:
                gray = int(img[i, j])
                color = (gray, gray, gray)
                cv.circle(pontos, (j, i), radius_borda, color, -1, cv.LINE_AA)
    res = np.concatenate((bordas, img, pontos), axis=1)
    #cv.imshow('Bordas', bordas)
    cv.imshow('Pontos', res)

def mudoustep(x):
    global step
    step = x
    mudarange()


def mudoujitter(x):
    global jitter
    jitter = x
    desenha()


def mudouradius(x):
    global radius
    radius = x
    desenha()


def mudouradius2(x):
    global radius_borda
    radius_borda = x
    desenha()


# cv.createTrackbar('Step', 'Pontos', step, step_max, mudoustep)
# cv.createTrackbar('Jitter', 'Pontos', jitter, jitter_max, mudoujitter)
# cv.createTrackbar('Radius', 'Pontos', radius, radius_max, mudouradius)
# cv.createTrackbar('RadiusB', 'Pontos', radius_borda, radius_borda_max, mudouradius2)
desenha()
cv.waitKey()
{% endhighlight %}

## Descrição do programa cannybordas.py

