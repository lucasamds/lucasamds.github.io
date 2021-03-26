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
            Duv = ((i - dft_height/2) ** 2 + (j - dft_width/2) ** 2)**0.5
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
    cv.imshow('Filtrada', filtrada)
{% endhighlight %}

Antes de fazer a filtragem é necessário criar o filtro, vamos falar sobre a `criafiltro()` mais pra frente, assim como a imagem de entrada, é preciso criar uma versão complexa do filtro, para isso criamos uma lista para guardar a parte real e imaginária do filtro, ambas terão os mesmos valores, com o `merge()` fazemos a união das partes. Para aplicar o filtro vamos utilizar a função `mulSpectrums()`, esta função realiza uma multiplicação ponto a ponto de dois espectros de Fourier, seus parâmetros são: primeira imagem que será utilizada; segunda imagem que será utilizada, precisa ser do mesmo tamanho da primeira, uma flag de operação, como não iremos utilizar nenhuma flag em específico o valor 0 foi passado.

Após realizar a filtragem damos início ao processo inverso, primeiramente voltamos os quadrantes da imagem para suas posições originais com mais uma chamada da função `trocaquadrantes()`, com isso já podemos calcular a transformada inversa, o resultado deste processo também será uma variável complexa, devido a isso é necessária fazer a separação dos canais, aqui estamos interessados apenas na parte real. A linha `filtrada = filtrada[0:height, 0:width]` recupera o tamanho original da imagem, lembre que nós calculamos um novo tamanho para a aplicação da DFT e criamos bordas para alcançar esse tamanho, aqui estamos removendo estas bordas.

{% highlight pyhton %}
def criafiltro(dft_height, dft_width):
    filter = np.empty((dft_height, dft_width), np.float32)
    for i in range(dft_height):
        for j in range(dft_width):
            Duv = ((i - dft_height/2) ** 2 + (j - dft_width/2) ** 2)**0.5
            filter[i, j] = (gammah - gammal)*(1-np.exp(-c*((Duv**2)/(radius**2)))) + gammal
    return filter
{% endhighlight %}

O filtro homomórfico que vamos utilizar segue a seguinte função:

$$
    H(u,v)=(\gamma _{H}-\gamma_{L})\left [ 1-e^{-c\left [ D^{2}(u,v)/D^{2}_{0} \right ]}  \right ]+\gamma_{L}
$$

Onde $$ \gamma_{H} $$ e $$ \gamma_{L} $$ são os parâmetros de limite do filtro e $$ c $$ indica a intensidade de transição entre elas; $$ D_{0} $$ é o raio do filtro utilizado e $$ D(u, v) $$ é um filtro passa-baixa ideal definido pela função abaixo:

$$
    D(u,v)=\left [ (u-\frac{P}{2})^2 + (v-\frac{Q}{2})^2 \right ]^{\frac{1}{2}}
$$

A **Figura 3** mostra uma imagem de um filtro gerado pela função `criafiltro()`.

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/filtro_homomorfico.png)
*Figura 3. Filtro homomórfico.*

{% highlight pyhton %}
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
{% endhighlight %}

As demais funções serão chamadas apenas quando houver alguma mudança nas barras de controle do programa, a lógica dela é atualizar o valor da variável que foi modificada e em seguida refazer a aplicação do filtro com os novos dados.

Em seguida um vídeo mostrando o funcionamento do programa *freqfilter.py* e algumas imagens filtradas pelo programa.

<iframe src="https://www.youtube.com/embed/kTWa1j_Gyeg?vq=hd1080&modestbranding=1&showinfo=0&rel=0&iv_load_policy=3" width="560" height="315" frameborder="0"></iframe>
<em class="descricao">Vídeo 1. Funcionamento do freqfilter.py</em>

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/saidafreq1.png)
![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/saidafreq2.png)
![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/saidafreq3.png)
*Figuras 4, 5 e 6. Exemplos de saída do programa freqfilter.py.*

Com isso encerramos mais um post desta série, até uma próxima!