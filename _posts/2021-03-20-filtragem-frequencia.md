---
layout: post
title: Filtragem no domínio da frequência
mathjax: true
---


<div class="message">
  Sexto post da série de processamento digital de imagens, hoje iremos filtar imagens no domínio da frequência.
</div>

Até então realizamos algumas filtragens no domínio espacial, para que possamos filtrar uma imagem no domínio da frequência é necessário primeiro realizar a *Transformada de Fourier*, para que tenhamos assim a imagem representada no domínio da frequência, no nosso caso, como estamos trabalhando com imagens digitais, iremos aplicar a *Transformada Discreta de Fourier*. A representação frequencial da imagem vai nos possibilitar perceber algumas degradações da imagem que seriam dificilmente notadas no domínio espacial, um exemplo seria um padrão que se repete por toda a imagem, tipicamente encontrado em fotos antigas.

## Filtro homomórfico

No experimento de hoje queremos pegar uma imagem que apresente uma má iluminação e em seguida tentar "equilibrar" a cena, para isso iremos utilizar um filtro homomórfico, este por sua vez é utilizado para aprimorar a visualização da imagem com base na iluminância e reflectância da cena. A ideia será, partindo da imagem de entrada, vamos calcular sua **DFT** e em seguida aplicar o filtro no resultado da transformada, por fim devemos realizar a transformada inversa para que seja possível visualizar o efeito da filtragem na imagem.


<a id="listagem1"></a>
##### Listagem 1. freqfilter.py
{% highlight python %}

import cv2 as cv
import numpy as np
import sys

img = cv.imread('imagens/dft.jpg', cv.IMREAD_GRAYSCALE)
if img is None:
    sys.exit("O arquivo não foi encontrado ou não existe")

width = len(img[0])//2
height = len(img)//2
img = cv.resize(img, (width, height), interpolation=cv.INTER_AREA)
gammal = 0
gammal_max = 20
gammah = 2
gammah_max = 50
c = 1
c_max = 100
radius = 5
radius_max = 500


def trocaquadrantes(img, height, width):
    img = img[0:(height & -2), 0:(width & -2)]
    cx = height // 2
    cy = width // 2
    q0 = img[0:cx, 0:cy]
    q1 = img[cx:cx + cx, 0:cy]
    q2 = img[0:cx, cy:cy + cy]
    q3 = img[cx:cx + cx, cy:cy + cy]
    tmp = np.copy(q0)
    img[0:cx, 0:cy] = q3
    img[cx:cx + cx, cy:cy + cy] = tmp
    tmp = np.copy(q1)
    img[cx:cx + cx, 0:cy] = q2
    img[0:cx, cy:cy + cy] = tmp
    return img


def mudouradius(x):
    global radius
    radius = x if x > 0 else 1
    fazdft(imgComplex)


def mudougammal(x):
    global gammal
    gammal = 0.2 * x
    fazdft(imgComplex)


def mudougammah(x):
    global gammah
    gammah = 0.2 * x
    fazdft(imgComplex)


def mudouc(x):
    global c
    c = x
    fazdft(imgComplex)


def criafiltro(dft_height, dft_width):
    filter = np.empty((dft_height, dft_width), np.float32)
    for i in range(dft_height):
        for j in range(dft_width):
            Duv = (i - dft_height/2) ** 2 + (j - dft_width/2) ** 2
            filter[i, j] = (gammah - gammal)*(1-np.exp(-c*((Duv**2)/(radius**2)))) + gammal
    return filter


def mostradft(imgComplex):
    planes = cv.split(imgComplex)
    planes[0] = cv.magnitude(planes[0], planes[1])
    imgModulo = planes[0]
    imgModulo = cv.add(np.ones(imgModulo.shape, imgModulo.dtype), imgModulo)
    imgModulo = cv.log(imgModulo)
    cv.normalize(imgModulo, imgModulo, 0, 1, cv.NORM_MINMAX)
    cv.imshow('DFT', imgModulo)


def fazfiltragem(imgComplex):
    filter = criafiltro(dft_height, dft_width)
    comps = [filter, filter]
    filter = cv.merge(comps)
    filtrada = cv.mulSpectrums(imgComplex, filter, 0)
    filtrada = trocaquadrantes(filtrada, dft_height, dft_width)
    filtrada = cv.idft(filtrada)
    planes.clear()
    planos = cv.split(filtrada)
    filtrada = planos[0]
    cv.normalize(filtrada, filtrada, 0, 1, cv.NORM_MINMAX)
    filtrada = filtrada[0:height, 0:width]
    cv.imshow('Original', img)
    #cv.imshow('Bordas', padded)
    cv.imshow('Filtrada', filtrada)


dft_width = cv.getOptimalDFTSize(width)
dft_height = cv.getOptimalDFTSize(height)
padded = cv.copyMakeBorder(img, 0, dft_height-height, 0, dft_width-width, cv.BORDER_CONSTANT, value=[0, 0, 0])

planes = [np.float32(padded), np.zeros(padded.shape, np.float32)]
imgComplex = cv.merge(planes)
imgComplex = cv.dft(imgComplex)
imgComplex = trocaquadrantes(imgComplex, dft_height, dft_width)
mostradft(imgComplex)
fazfiltragem(imgComplex)

cv.namedWindow('Original', cv.WINDOW_AUTOSIZE)
cv.namedWindow('Filtrada', cv.WINDOW_AUTOSIZE)
cv.createTrackbar('radius', 'Filtrada', radius, radius_max, mudouradius)
cv.createTrackbar('gammal', 'Filtrada', gammal, gammal_max, mudougammal)
cv.createTrackbar('gammah', 'Filtrada', gammah, gammah_max, mudougammah)
cv.createTrackbar('c', 'Filtrada', c, c_max, mudouc)

cv.waitKey()

{% endhighlight %}

## Descrição do programa freqfilter.py

{% highlight python %}
img = cv.imread('imagens/dft.jpg', cv.IMREAD_GRAYSCALE)
if img is None:
    sys.exit("O arquivo não foi encontrado ou não existe")

width = len(img[0])//2
height = len(img)//2
img = cv.resize(img, (width, height), interpolation=cv.INTER_AREA)
gammal = 0
gammal_max = 20
gammah = 2
gammah_max = 50
c = 1
c_max = 100
radius = 5
radius_max = 500
{% endhighlight %}

Assim como nos nossos experimentos anteriores, iniciamos o programa fazendo a leitura da imagem que iremos realizar a filtragem, também é feita a declaração de algumas variáveis de controle, iremos falar mais um pouco sobre elas mais pra frente, note que a imagem foi redimensionada apenas para facilitar sua visualização durante a execução do programa. 

{% highlight python %}

dft_width = cv.getOptimalDFTSize(width)
dft_height = cv.getOptimalDFTSize(height)
padded = cv.copyMakeBorder(img, 0, dft_height-height, 0, dft_width-width, cv.BORDER_CONSTANT, value=[0, 0, 0])

{% endhighlight %}

A performance da DFT vai depender da dimensão da imagem, uma matriz que possui tamanho com seu valor sendo uma potência de 2 será processada bem mais rápida, por exemplo. Para melhorar o desempenho dos cálculos, vamos utilizar a função `getOptimalDFTSize()`, esta por sua vez irá receber um inteiro, que representa o tamanho de um vetor, e retornar o próximo valor ótimo após o tamanho passado; com isso teremos um novo valor para largura e altura da imagem. 

O próximo passo é preencher a imagem original com bordas até que ela chegue nos novos tamanhos que foram calculados, para esta tarefa vamos utilzar a função `copyMakeBorder()`, ela vai receber os seguintes parâmetros: imagem em que serão adicionadas as bordas; tamanho da borda de cima; tamanho da borda de baixo; tamanho da borda da esquerda; tamanho da borda da direita; tipo da borda; cor da borda. Estamos adicionando bordas apenas na direita e na parte de baixo da figura, a cor escolhida foi o preto.

{% highlight python %}
planes = [np.float32(padded), np.zeros(padded.shape, np.float32)]
imgComplex = cv.merge(planes)
imgComplex = cv.dft(imgComplex)
imgComplex = trocaquadrantes(imgComplex, dft_height, dft_width)
mostradft(imgComplex)
fazfiltragem(imgComplex)
{% endhighlight %}

Após redimensionar a imagem, criamos sua versão complexa, onde a parte real é a própria imagem e a parte imaginária é compostas por zeros, a uninão das duas partes é feita pela função `merge()`, esta função recebe uma lista de vetores de único canal e retorna um vetor multicanal que contém todos eles. Com a matriz complexa formada, realizamos o cálculo da DFT.

Antes de aplicar o filtro no domínio da frequência vamos utilizar a propriedade da periodicidade da DFT, fazendo uma troca nos quadrantes da imagem, com isso os pontos de interesse estarão concentrados no centro da imagem. A troca é feita da seguinte maneira:

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/quadrantes.png)
*Figura 1. Troca de quadrantes*

Agora que tudo está preparado, a função `mostradft()` mostra em tela o módulo do resultado da DFT, já com os quadrantes trocados, e por fim realizamos a aplicação do filtro com a `fazfiltragem()`.

{% highlight python %}
def trocaquadrantes(img, height, width):
    img = img[0:(height & -2), 0:(width & -2)]
    cx = height // 2
    cy = width // 2
    q0 = img[0:cx, 0:cy]
    q1 = img[cx:cx + cx, 0:cy]
    q2 = img[0:cx, cy:cy + cy]
    q3 = img[cx:cx + cx, cy:cy + cy]
    tmp = np.copy(q0)
    img[0:cx, 0:cy] = q3
    img[cx:cx + cx, cy:cy + cy] = tmp
    tmp = np.copy(q1)
    img[cx:cx + cx, 0:cy] = q2
    img[0:cx, cy:cy + cy] = tmp
    return img
{% endhighlight %}

Para realizar a troca de quadrantes a função recebe a imagem que se deseja fazer a troca e suas dimensões, a linha `img = img[0:(height & -2), 0:(width & -2)]` garante que a imagem terá tamanho par, fazendo um *AND* lógico com -2; as variáveis `q0`, `q1`, `q2` e `q3` vão copiar os quadrantes da imagem e em seguida serão utilizadas para realizar as trocas necessárias, por fim a função retorna a nova imagem, com os quadrantes trocados.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/dftmodulo.png)
*Figura 2. Imagem antes e depois da troca de quadrantes.*

{% highlight python %}
def mostradft(imgComplex):
    planes = cv.split(imgComplex)
    planes[0] = cv.magnitude(planes[0], planes[1])
    imgModulo = planes[0]
    imgModulo = cv.add(np.ones(imgModulo.shape, imgModulo.dtype), imgModulo)
    imgModulo = cv.log(imgModulo)
    cv.normalize(imgModulo, imgModulo, 0, 1, cv.NORM_MINMAX)
    cv.imshow('DFT', imgModulo)
{% endhighlight %}

A função `mostradft()` vai preparar a imagem complexa para ser impressa em tela. A primeira coisa que ela faz é a separação dos canais com a função `split()`, aqui estamos interessados apenas na magnitude da nossa matriz complexa. Como os valores resultantes da DFT podem ter uma discrepância muito grange, para que possavamos perceber esses valores na imagem que será mostrada na tela, vamos mudar os valores para a escala logarítma, note que antes de calcular o logarítmo dos valores adicionamos uma unidade em cada elemento, assim evitando uma indeterminação pelo `log(0)`. Como de costume, normalizamos os valores da imagem antes de apresentá-la.

{% highlight python %}
def fazfiltragem(imgComplex):
    filter = criafiltro(dft_height, dft_width)
    comps = [filter, filter]
    filter = cv.merge(comps)
    filtrada = cv.mulSpectrums(imgComplex, filter, 0)
    filtrada = trocaquadrantes(filtrada, dft_height, dft_width)
    filtrada = cv.idft(filtrada)
    planes.clear()
    planos = cv.split(filtrada)
    filtrada = planos[0]
    cv.normalize(filtrada, filtrada, 0, 1, cv.NORM_MINMAX)
    filtrada = filtrada[0:height, 0:width]
    cv.imshow('Original', img)
    #cv.imshow('Bordas', padded)
    cv.imshow('Filtrada', filtrada)
{% endhighlight %}

Antes de fazer a filtragem é necessário criar o filtro, vamos falar sobre a `criafiltro()` mais pra frente, assim como a imagem de entrada, é preciso criar uma versão complexa do filtro, para isso criamos uma lista para guardar a parte real e imaginária do filtro, ambas terão os mesmos valores, com o `merge()` fazemos a união das partes. Para aplicar o filtro vamos utilizar a função `mulSpectrums()`, esta função realiza uma multiplicação ponto a ponto de dois espectros de Fourier, seus parâmetros são: primeira imagem que será utilizada; segunda imagem que será utilizada, precisa ser do mesmo tamanho da primeira, uma flag de operação, como não iremos utilizar nenhuma flag em específico o valor 0 foi passado.

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