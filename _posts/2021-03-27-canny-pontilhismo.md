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

Neste experimento queremos utilizar o algoritmo de Canny para criar uma arte digital. Primeiramente vamos criar uma imagem utilizando o método de pontilhismo, está técnica consiste em criar cenas utilizando apenas pontos. Após criar a imagem pontilhada, vamos detectar as bordas da cena original e fazer com que seja desenhado um ponto em cada pixel foi detectado como borda.

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
step = 4
step_max = 50
jitter = 30
jitter_max = 100
radius = 4
radius_max = 100
radius_borda = 1
radius_borda_max=10

#img = cv.resize(img, (len(img[0])//2, len(img)//2), interpolation=cv.INTER_AREA)
#cv.imshow('Original', img)
height = img.shape[0]
width = img.shape[1]
cv.namedWindow('Pontos', cv.WINDOW_AUTOSIZE)

th1 = 65
th1_max = 200
th2 = 2*th1


def setrange():
    global xrange, yrange
    xrange = np.array(range(0, height // step))
    yrange = np.array(range(0, width // step))
    for i in range(0, len(xrange)):
        xrange[i] = xrange[i]*step+step//2

    for i in range(0, len(yrange)):
        yrange[i] = yrange[i]*step+step//2
    desenha()


def desenha():
    global pontos
    pontos = np.full((height, width), 255, dtype=np.uint8)
    random.shuffle(xrange)
    for i in range(0, xrange.shape[0]):
        random.shuffle(yrange)
        for j in range(0, yrange.shape[0]):
            x = xrange[i] + random.randint(0, (2 * jitter)) - jitter + 1
            if x >= height:
                x = height-1
            y = yrange[j] + random.randint(0, (2 * jitter)) - jitter + 1
            if y >= width:
                y = width-1
            gray = int(img[x, y])
            color = (gray, gray, gray)
            cv.circle(pontos, (y, x), radius, color, -1, cv.LINE_AA)
    fazbordas()


def fazbordas():
    bordas = cv.Canny(img, th1, th2)
    for i in range(0, height):
        for j in range(0, width):
            if bordas[i, j] == 255:
                gray = int(img[i, j])
                color = (gray, gray, gray)
                cv.circle(pontos, (j, i), radius_borda, color, -1, cv.LINE_AA)
    res = np.concatenate((bordas, img, pontos), axis=0)
    #cv.imwrite('imagens/resultado_pontos1.png', res)
    #cv.imshow('Bordas', bordas)
    cv.imshow('Pontos', res)

def mudoustep(x):
    global step
    step = x
    setrange()


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


def mudouth1(x):
    global th1, th2, bordas
    th1 = x
    th2 = 2*th1
    bordas = cv.Canny(img, th1, th2)
    desenha()


cv.createTrackbar('Step', 'Pontos', step, step_max, mudoustep)
cv.createTrackbar('Jitter', 'Pontos', jitter, jitter_max, mudoujitter)
cv.createTrackbar('Radius', 'Pontos', radius, radius_max, mudouradius)
cv.createTrackbar('th1', 'Pontos', th1, th1_max, mudouth1)
cv.createTrackbar('RadiusB', 'Pontos', radius_borda, radius_borda_max, mudouradius2)
setrange()
cv.waitKey()
{% endhighlight %}

## Descrição do programa cannybordas.py

O programa vai seguir o seguinte fluxo, primeiro configuramos os vetores que irão fazer a amostragem da imagem, em seguida chamamos a função que desenha os pontos na cena que estamos criando, com os pontos já criados, adicionamos as bordas que foram detectadas por cima da imagem feita.
{% highlight python %}

def setrange():
    global xrange, yrange
    xrange = np.array(range(0, height // step))
    yrange = np.array(range(0, width // step))
    for i in range(0, len(xrange)):
        xrange[i] = xrange[i]*step+step//2

    for i in range(0, len(yrange)):
        yrange[i] = yrange[i]*step+step//2
    desenha()


{% endhighlight %}

A primeira função na sequência do processamento é a `setrange()`, os *Arrays* `xrange` e `yrange` serão responsáveis por dizer quais posições da imagem original serão amostrados, como não queremos pegar todos os pontos, seus tamanhos serão frações das dimensões da imagem base; após criados eles recebem um ganho de valor `step` e um deslocamento `step//2`, com isso fazemos com que o pocesso de amostragem ocorra no centro da janela.

{% highlight python %}
def desenha():
    global pontos
    pontos = np.full((height, width), 255, dtype=np.uint8)
    random.shuffle(xrange)
    for i in range(0, xrange.shape[0]):
        random.shuffle(yrange)
        for j in range(0, yrange.shape[0]):
            x = xrange[i] + random.randint(0, (2 * jitter)) - jitter + 1
            if x >= height:
                x = height-1
            y = yrange[j] + random.randint(0, (2 * jitter)) - jitter + 1
            if y >= width:
                y = width-1
            gray = int(img[x, y])
            color = (gray, gray, gray)
            cv.circle(pontos, (y, x), radius, color, -1, cv.LINE_AA)
    fazbordas()
{% endhighlight %}

Em seguida temos a função `desenha()`, ela quem será responsável por criar os pontos na nossa imagem de saída. Iniciamos com a criação da matriz `pontos`, todos as posições da matriz recebem o valor **255**, resultando em uma cena toda branca. Com o fundo da imagem criado, damos início ao processo de preenchimento, para manter uma certa aleatoriedade no posicionamento dos pontos amostrados, fazemos embaralhamentos nos elementos de `xrange` e `yrange`. Antes de desenhar os pontos, vamos adicionar mais um deslocamento aleatório em ambas as direções de cada amostra, este deslocamento é feito com base no valor da variável `jitter`, os valores de coordenadas resultantes indicam em qual posição da imagem base devemos buscar a cor que o ponto a ser desenhado terá.

Para desenhar os pontos estamos utilizando a função `circle()` do OpenCV, seus parâmetros de entrada são: imagem que o círculo deve ser desenhado; coordenadas de origem do círculo; raio do círculo; cor da borda do círculo; espessura da borda do círculo, caso o valor seja -1 a função preenche o círculo com a cor da borda; tipo de linha utilizada, estamos utilizando o tipo que opera com *antialiasing*, fazendo com que os círculos não tenham bordas serrilhadas.

{% highlight python %}
def fazbordas():
    bordas = cv.Canny(img, th1, th2)
    for i in range(0, height):
        for j in range(0, width):
            if bordas[i, j] == 255:
                gray = int(img[i, j])
                color = (gray, gray, gray)
                cv.circle(pontos, (j, i), radius_borda, color, -1, cv.LINE_AA)
    res = np.concatenate((bordas, img, pontos), axis=0)
    #cv.imwrite('imagens/resultado_pontos1.png', res)
    #cv.imshow('Bordas', bordas)
    cv.imshow('Pontos', res)
{% endhighlight %}

Com os pontos formados, nos resta agora preencher os espaços que representam bordas na imagem base. Para tal, vamos primeiro criar uma imagem que contém apenas as bordas encontradas pelo algoritmo de Canny, a função `Canny()` faz as operações de filtragem e nos devolve a imagem resultante, a função recebe a figura a ser filtrada e os dois limiares que serão utilizados no processo. Para criar as bordas na cena pontilhada, vamos percorrer toda a matriz de bordas, como esta matriz possui apenas valores iguais a **0** ou **255**, ao encontrar o valor **255** significa que aquele pixel faz parte de uma borda, pegamos então a cor que esse pixel possui e em seguida criamos um novo ponto na imagem final, de mesma posição e cor. O restante da função apenas prepara o conteúdi para ser apresentado em tela. Abaixo um comparativo entre uma imagem criada sem a adição de bordas e com a adição de bordas.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/sborda_cborda.png)
*Figura 3. Figura pontilhada sem adição de bordas x com adição de bordas*

Como é possível perceber, a inclusão das bordas faz uma diferença significativa na visualização da imagem. Veja alguns resultados do programa *cannybordas.py*.

[![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/resultado_pontos1.png)](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/resultado_pontos1.png)
