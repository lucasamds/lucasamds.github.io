---
layout: post
title: Filtragem no domínio espacial II
mathjax: true
---


<div class="message">
  Quinto post da série de processamento digital de imagens com Python e OpenCV, desta vez vamos nos aventurar com o efeito fotográfico tilt-shift.
</div>

## Aplicação em imagens

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
alt_slider_max = height * 2
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
    global l1, l2, alt
    l1 += (x - alt) // 2
    l2 += (x - alt) // 2
    alt = x
    criaPond(l1, l2, d)


def mudameio(x):
    global meio, l1, l2
    l1 -= x-meio
    l2 += x-meio
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

{% highlight python %}
d_slider_max = 100
alt_slider_max = height * 2
meio_slider_max = height
tbd = 'd'
tbalt = 'altura'
tbm = 'meio'
title_window = 'tilt-shift'
cv.namedWindow(title_window)
{% endhighlight %}

Em seguida criamos algumas variáveis que serão utilizadas na criação do *Trackbar*, neste programa teremos três barras de calibração, a primeira irá controlar o valor da variável `d`, esta variável é responsável pela *força* de transição entre a imagem original e a imagem borrada, quanto maior for o valor de `d` mais suave é a transição entre as imagens; a segunda barra controla a *altura* da região de foco, quanto maior a altura maior será a área em destaque; a terceira e última barra controla a posição do centro da região de foco, no sentido do eixo vertical, quando maior seu valor mais para baixo o centro será deslocado. Em seguida teremos algumas funções que vão ser responsáveis por implementar o efeito na imagem.

{% highlight python %}
def pondera(x, l1, l2, d):
    if d == 0:
        d = 1
    return (np.tanh((x+l1)/d) - np.tanh((x-l2)/d))/2
{% endhighlight %}

A função `pondera()` vai calcular o valor que deve multiplicar cada elemento de uma determinada linha `x` da imagem original, o cálculo é dado pela seguinte função:

$$
    \alpha (x)=\frac{1}{2}(\tanh \frac{x-l1}{d} - \tanh \frac{x-l2}{d})
$$

Como já foi dito, **d** indica a força com que a ponderação aumenta, **l1** vai representar a posição de corte superior da zona de foco e **l2** a posição de corte inferior.

{% highlight python %}
def criaPond(l1, l2, d):
    global img1
    for i in range(height):
        alpha = pondera(i, l1, l2, d)
        img1[i] = cv.addWeighted(img[i], alpha, img2[i], 1-alpha, 0)
    img1 = np.uint8(img1)
    cv.imshow(title_window, img1)
{% endhighlight %}

A função principal do nosso programa será a `criaPond()`, aqui iremos calcular um valor de `alpha` par cada linha da imagem, utilizando a função anterior, já que para cada linha teremos um resultado diferente; calculando o alpha, temos agora que criar a imagem que será nosso resultado final, esta imagem será uma soma ponderada entre a original e sua versão borrada. A função `addWeighted()` recebe as duas imagens que serão adicionadas e seus respectivos pesos, ela faz o cálculo da nossa imagem resultante da seguinte maneira:

$$
    res = img_{orig}\times \alpha + img_{blur}\times (1-\alpha)+\gamma
$$

Note que neste exemplo estamos utilizando o valor de *gamma* igual a 0. Antes de apresentar o resultado é necessário converter a imagem para o formato `np.uint8`, limitando assim os valores de cada canal entre 0 e 255.

{% highlight python %}
def mudad(x):
    global d
    d = x
    criaPond(l1, l2, d)


def mudaalt(x):
    global l1, l2, alt
    l1 += (x - alt) // 2
    l2 += (x - alt) // 2
    alt = x
    criaPond(l1, l2, d)


def mudameio(x):
    global meio, l1, l2
    l1 -= x-meio
    l2 += x-meio
    meio = x
    criaPond(l1, l2, d)
{% endhighlight %}

As funções acima são chamadas apenas quando existe uma mudança no valor das variáveis `d`, `alt` ou `meio`, através de seus respctivos *Trackbars*, nelas são realizadas as atualizações de valores necessárias para o novo cálculo da imagem resultante.

{% highlight python %}
criaPond(l1, l2, d)
cv.createTrackbar(tbd, title_window, d, d_slider_max, mudad)
cv.createTrackbar(tbalt, title_window, alt, alt_slider_max, mudaalt)
cv.createTrackbar(tbm, title_window, meio, alt_slider_max, mudameio)

key = cv.waitKey()
if key == ord('s'):
    cv.imwrite('imagens/resultado_tiltshift.png', img1)

cv.destroyAllWindows()
sys.exit()

{% endhighlight %}

A parte final do código vai gerar uma imagem inicial e criar as barras de controle do efeito. A função `createTrackbar()` recebe os seguintes parâmetros: uma *flag* que denomina a barra; o nome da janela em que a barra será criada; a variável que servirá como referência para o valor inicial da barra; o valor máximo da barra; a função que deve ser chamada na mudança de valor. Por fim, o programa aguarda indefinidamente por um acionamento do teclado, caso a tecla pressionada seja a letra **s**, o programa salva a imagem resultante antes do encerramento.

<iframe src="https://www.youtube.com/embed/sQjc0EB8yUo?vq=hd1080&modestbranding=1&showinfo=0&rel=0&iv_load_policy=3" width="560" height="315" frameborder="0"></iframe>
<em class="descricao">Vídeo 1. Funcionamento do programa tiltshift.py</em>

Abaixo alguns exemplos de saída do programa.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/resultado_tiltshift1.png)
![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/resultado_tiltshift2.png)
![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/resultado_tiltshift4.png)
![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/resultado_tiltshift3.png)

## Aplicação em vídeo

Agora que já falamos um pouco em como implementar o efeito em figuras, vamos tentar extender um pouco o problema e aplicar o *tilt-shift* em um vídeo, queremos também deixar o vídeo com o aspecto de uma sequência em stop motion.

##### Listagem 2. tiltshiftvideo.py

{% highlight python %}
import cv2 as cv
import numpy as np

cap = cv.VideoCapture('videos/vid2.mp4')
width = int(cap.get(3))
height = int(cap.get(4))
frame = np.empty((height, width, 3))
quadro = 1
width = width//2
height = height//2
d = 20
l1 = -height//4
l2 = height//2 - l1

alt = l2 + l1

meio = l2-(alt//2)
media = np.ones([13, 13], np.float32)/169

d_slider_max = 100
alt_slider_max = height * 2
meio_slider_max = height
tbd = 'd'
tbalt = 'altura'
tbm = 'meio'
title_window = 'tilt-shift'
title_window2 = 'Original'
cv.namedWindow(title_window)
#cv.namedWindow(title_window2)

def mudad(x):
    global d
    d = x
    criaPond(l1, l2, d)


def mudaalt(x):
    global l1, l2, alt
    l1 += (x - alt) // 2
    l2 += (x - alt) // 2
    alt = x
    criaPond(l1, l2, d)


def mudameio(x):
    global meio, l1, l2
    l1 -= x - meio
    l2 += x - meio
    meio = x
    criaPond(l1, l2, d)


def criaPond(l1, l2, d):
    global img1
    for i in range(height):
        alpha = pondera(i, l1, l2, d)
        img1[i] = cv.addWeighted(frame[i], alpha, img2[i], 1-alpha, 0)
    img1 = np.uint8(img1)
    cv.imshow(title_window, img1)
    out.write(img1)


def pondera(x, l1, l2, d):
    if d == 0:
        d = 1
    return (np.tanh((x+l1)/d) - np.tanh((x-l2)/d))/2

fourcc = cv.VideoWriter_fourcc('M', 'J', 'P', 'G')
out = cv.VideoWriter('videos/saida.avi', fourcc, 9, (width,height))
while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        frame = cv.resize(frame, (width, height), interpolation=cv.INTER_AREA)
        #cv.imshow(title_window2, frame)
        quadro += 12
        cap.set(1, quadro)
        img1 = np.empty((height, width, 3))
        img2 = cv.filter2D(frame, -1, media)
        criaPond(l1, l2, d)
        cv.createTrackbar(tbd, title_window, d, d_slider_max, mudad)
        cv.createTrackbar(tbalt, title_window, alt, alt_slider_max, mudaalt)
        cv.createTrackbar(tbm, title_window, meio, alt_slider_max, mudameio)
        cv.waitKey(9)

    else:
        break

cap.release()
out.release()
cv.destroyAllWindows()
{% endhighlight %}

## Descrição do programa tiltshiftvideo.py

O segundo código usa a <a href="#listagem1">Listagem 1</a> como base, mudando apenas alguns parâmetros inicialmente, aqui vamos utilizar o `VideoCapture()` no lugar do `imread()`, visto que estamos interessados em abrir um arquivo de vídeo. A máscara do filtro da média também sofreu uma leve alteração, ampliamos seu tamanho para uma matriz $$ 13 \times 13 $$.

{% highlight python %}

fourcc = cv.VideoWriter_fourcc('M', 'J', 'P', 'G')
out = cv.VideoWriter('videos/saida.avi', fourcc, 9, (width,height))

{% endhighlight %}

Neste trecho do código criamos o objeto `out`, responsável pela gravação do vídeo que iremos gerar, para isso precisamos primeiro gerar o código do **FourCC**, este por sua vez é um código de 4 bytes que especifíca como vai funcionar a codificação/decodificação do vídeo que iremos criar, aqui foi escolhido o modelo *'MJPG'*. O objeto `out` será do tipo `VideoWriter`, na criação ele recebe o diretório de onde deve salvar o vídeo de saída, o FourCC, a taxa de frames por segundo que o vídeo deve apresentar e as dimensões do arquivo.

{% highlight python %}

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        frame = cv.resize(frame, (width, height), interpolation=cv.INTER_AREA)
        #cv.imshow(title_window2, frame)
        quadro += 12
        cap.set(1, quadro)
        img1 = np.empty((height, width, 3))
        img2 = cv.filter2D(frame, -1, media)
        criaPond(l1, l2, d)
        cv.createTrackbar(tbd, title_window, d, d_slider_max, mudad)
        cv.createTrackbar(tbalt, title_window, alt, alt_slider_max, mudaalt)
        cv.createTrackbar(tbm, title_window, meio, alt_slider_max, mudameio)
        cv.waitKey(9)

    else:
        break

{% endhighlight %}

Como no caso do vídeo queremos aplicar o efeito do *tilt-shift* para diversas imagens seguidas, vamos utilizar um laço que funciona baseado no tamanho do arquivo, enquanto o programa conseguir capturar novas cenas, ficamos no laço. Iniciamos o processo fazendo uma redimensão da imagem capturada, já que o arquivo em questão apresentava grandes dimensões, este passo é opcional; em seguida fazemos com que o programa avance em 12 posições o índice do quadro que será lido em seguida, por meio do método `set()`, com isso nós vamos receber um quadro, realizar todas as operações que geram o efeito do tilt-shift, ignorar os próximos 11 quadros e repetir o processo; este salto de frames é o que simula o efeito de *stop motion* na cena.

A criação das barras de controle do efeito é feita da mesma forma do código anterior, as funções utilizadas também são iguais ao da Listagem 1, com uma pequena mudança apenas em `criaPond()`.

{% highlight python %}

def criaPond(l1, l2, d):
    global img1
    for i in range(height):
        alpha = pondera(i, l1, l2, d)
        img1[i] = cv.addWeighted(frame[i], alpha, img2[i], 1-alpha, 0)
    img1 = np.uint8(img1)
    cv.imshow(title_window, img1)
    out.write(img1)

{% endhighlight %}

Após apresentar o resultado em tela, o programa chama o método `write()` do objeto de saída, passando como parâmetro a imagem resultante do efeito. Abaixo alguns exemplos de saída do programa.

<iframe src="https://www.youtube.com/embed/3W_CuVXHHwg?vq=hd1080&showinfo=0&rel=0&iv_load_policy=3" width="560" height="315" frameborder="0"></iframe>

<iframe src="https://www.youtube.com/embed/Ok8ChnE9Dx0?modestbranding=1&showinfo=0&rel=0&iv_load_policy=3" width="560" height="315" frameborder="0"></iframe>

<iframe src="https://www.youtube.com/embed/qmPHJAh_-OE?vq=hd1080&showinfo=0&rel=0&iv_load_policy=3" width="560" height="315" frameborder="0"></iframe>
<em class="descricao">Vídeos 2, 3 e 4. Exemplos de saída do tiltshiftvideo.py</em>

A combinação dos efeitos de *tilt-shif* e *stop motion* geram um resultado bem interessante. Com isso encerramos mais um post, nos vemos numa próxima e bons estudos!