---
title: "Análise e visualização de dados espaciais com R - 1a. Parte"
author: "Walter Humberto Subiza Piña"
date: "2018-08-28"
output: 
  html_document:
    keep_md: true
---





### Indice

#### 1. Introdução

Objetivo: apresentar algumas formas de tratar e visualizar os dados georreferenciados no _R_.

Conceito de **feição** ou **_simple feature_**. Apresentação de pacotes `sf`, `raster` e `sp`.

#### 2. Objetos classe S-3 e S-4

  a- Definição
        
  b- Representação e geometria vetorial
  
  + Dimensão e coordenadas
      
  + organização das feições simples no `sf`
      
  + Métodos para cada classe
      
  + Tipos de geometria

  c- Dados matriciais ou raster
  
#### 3. Sistema Geodésico de Referência - SGR ( _CRS_)

 a- _CRS_ não projetado ou de coordenadas geográficas

 b- _CRS_ de coordenadas projetadas

  Tabelas de SGR no Brasil
  
     Exercício identificação de SGR

\
---
\

### 1- Introdução

 <div style="text-align:justify" markdown="1">
 
A inclusão da informação espacial nos dados estatísticos introduz um elemento qualitativo importante, tanto na etapa de análise, quanto na de disseminação da informação e tem-se tornado tema recorrente no ámbito do geoprocessamento e da estatística. Essa tendência global também se consolida a nível nacional, (criação da Infraestrutua Nacional de Dados Espaciais - INDE), assim como no IBGE, por exemplo, com a realização em 2016 de um Seminário de Metodologia que foi denominado “Integração de Dados Estatísticos e Geoespaciais e Visualização de Informações” ou ainda o planejamento do novo CensoGeo, previsto para o ano de 2022. 

A representação espacial se formaliza por meio do conceito de **feição simples** ou **_simple feature_**. A feição está definida na norma **ISO 19125-1:2004** que descreve como os objetos do mundo real podem ser representados em computadores, enfatizando o aspecto da geometria espacial. 

A norma estabelece também como esses elementos podem ser armazenados e recuperados de bases de dados e ainda que tipo de operações podem ser efetuadas. Esse padrão é amplamente usado e implementado em bases de dados (como PostGis) e software de  Sistemas de Informação Geográfica – SIG, como QGIS e ArcGis. Uma feição pode ser, por exemplo, uma casa, uma árvore ou uma estrada. Da mesma forma, um conjunto de feições pode-se considerar uma única feição, um conjunto de árvores pode ser uma floresta ou um conjunto de casas, uma vila ou cidade.

Esta documentação, de caráter introdutório, tem como **objetivo apresentar formas de tratar e visualizar os dados no _R_, considerando sua geometria e espacialidade **. Não tem a intenção de substituir o uso de SIG, principalmente na elaboração de mapas, cartogramas finais ou análises complexas, mas mostrar que é possível manipular e visualizar dados espaciais sem ter que, nesse processo, fazer o caminho do _R_ ao SIG e vice-versa várias vezes. Para isso serão apresentados dois pacotes específicos: `sf` e `raster`, além de alguns  para visualização. 

Especificamente, o pacote `sf` para dados vetoriais, facilita a leitura e manipulação se comparado a pacotes anteriores, como o `sp`, já que os objetos `sf` são dataframes com atributos espaciais e podem ser manipulados como tais usando pacotes como `tidyverse::dplyr`. Os objetos criados contém os metadados do SGR e guardam numa única coluna os atributos de georrefenciamento. A representação das feições é similar ao **PostGIS** e todas as funções comecam com o prefixo `st_`, em referência a espacio e tempo, facilitando assim sua búsqueda e uso.

O anterior pacote de manipulação de dados georreferenciados `sp` tem uma longa trajetória no sistema _R_ e está amplamente difundido sendo incluído como dependência em diversos pacotes, já o novo pacote `sf` está em fase de desenvolvimento, mas tem um futuro muito promissor enquanto mais simples de usar. Desta forma, é importante conhecer como é feita a conversão de formato entre os dois pacotes, a fim de aproveitar ao máximo os benefícios de ambos.

</div>

A transformação de um objeto `sf` em `sp` e viceversa, pode ser feita da seguinte forma:

  1- `sf` em `sp`, usando a função `as`:
  
    objeto_sp <- as(objeto_sf, Class = "Spatial")
  
  2- `sp` em `sf`, usando a função `st_as_sf`:
  
    objeto_sf <- st_as_sf(objeto_sp)
 
 Em relação aos dados matriciais ou _raster_, o pacote `raster` permite a manipulação e visualização de dados desse tipo. Vale salientar a aparição de um novo pacote `star`, em fase de desenvolvimento, que tratará todo tipo de dados espaço-temporais, vetoriais e matriciais, numa abordagem do tipo `tidyverse`.

Nesta primeira parte veremos alguns conceitos teóricos necessários para entender como se configuram e relacionam os dados com o espaço terrestre.

Nos próximos documentos trataremos de:

  . como importar dados vetoriais com o pacote `sf`.
  
  . como importar dados matriciais ou _raster_ com uma ou várias camadas, usando o pacote `raster`.
  
  . manipulação e transformação dos dados vetoriais e matriciais.
  
  . gravação de objetos vetoriais ou matriciais, com os pacotes `sf` e `raster`.
  
  . algumas funções de análisis de dados georreferenciados.
  
  . visualização de dados georreferenciados, com os pacotes `plot`, `tmap`, `ggmap`, `ggplot`.
 
\
--- 
\

### 2- Objetos classe S-3 e S-4

---

#### a- Definição

<div style="text-align:justify" markdown="1">

Para entender como os dados espaciais são tratados no _R_, precisamos ver que tipos de sistemas orientados a objeto suporta e em qual deles, esses dados se encaixam.

O _R_ tem várias classes de sistemas orientados a objeto, as mais usadas são S3 e S4, cada uma com sua definição, atributos e métodos específicos. 

Quando é criado um vetor, ou fazemos a importação de uma tabela, dataframe ou lista estamos criando objetos S3. O pacote `sf` define os objetos como do tipo S-3, facilitando assim sua manipulação e dos seus atributos. Mas específicamente os objetos `sf` são uma sub-classe da classe `data.frame` e da classe `tbl_df`.

Os objetos S4, usados nos dados raster ou matriciais, são derivados do S3 e possuem uma definição mais formal da classe. Em dados espaciais, os pacotes `raster` e `sp`criam objetos classe 4.

Na tabela seguint apresentamos as principais diferenças entre as classes S-3 e S-4

|Classe S-3             | Classe S-4              |
|-----------------------|-------------------------|
|Sem definição formal| Classe definida com setClass()|
|Objetos criados com atributo de classe|Objetos criados usando new() ou com pacotes específicos como `raster`|
|Acesso de atributos com \$|Acesso de atributos com \@ ou usando slots()|

</div>

---

#### b- Representação e geometria vetorial

<div style="text-align:justify" markdown="1">

Os objetos do mundo real precisam ser representados usando tipos diferentes de geometria. Toda geometria é composta de pontos simples da mesma classe, em diferentes espaços dimensionais. 
 
Assim sendo podemos representar os objetos usando três tiposbásicos  de geometria: **ponto, linha e polígono**. Um ponto pode ser representado numa superficie por dois atributos espacias, X e Y, que são suas coordenadas. Uma linha é uma sucessão de pontos e um polígono é uma sucessão de linhas que não se cruzam e fecham num determinado ponto inicial. No _R_ , é possível a combinação no mesmo objeto  de diversas geometrias como pontos, linhas e polígonos, o que não acontece nos SIG.

As geometrias mencionadas são representados num espacio bidimensional. Podemos incluir uma terceira e ou uma quarta dimensão, incluindo atributos como altitudes ou tempo por exemplo (dimensões Z ou M respectivamente). Assim sendo podemos representar o crescimento de uma árvore ao longo da sua vida usando as variáveis X e Y para dar a localização espacial, Z para registrar o crescimento em altura e M para estabelecer a data de cada medição.

Em resumo um **objeto pode ser representado espacialmente** por um conjunto de coordenadas com as seguintes caracteristicas:

  - X, Y  - localização espacial (bidimensional)
  
  - X, Y, Z  - localização espacial + altura ou altitude (tridimensional)
  
  - X, Y, M  - localização espacial + outra variavel medida (tridimensional)
  
  
  - X, Y, Z, M - localização espacial + altura ou altitude + outra variavel medida (quadridimensional)

---

No pacote `sf`, os principais tipos de representação da geometria de feiçoes simples  são:
 
  1- _**POINT**_ : geometria com dimensão zero contendo um único ponto, representado numa superfície por um par de coordenadas;
  
  2- _**LINESTRING**_: geometria unidimensional composta de uma sequencia de pontos (_**POINT**_), conectados por linhas que não se intersectam;
  
  3- _**POLYGON**_: geometria bidimensional com uma area positiva, formada por uma sequência de anéis de pontos fechados e não intersectados. O primeiro anel será o polígono exterior e os seguintes delimitan espaços vazios interiores, se eles existirem;
  
  4- _**MULTIPOINT**_: conjunto de pontos, chamado de simples se não tem 2 pontos iguais. Cada ponto possui seu par de coordenadas;
  
  5- _**MULTISTRING**_: conjunto de _**LINESTRING**_ ;
  
  6- _**MULTIPOLYGON**_: conjunto de _**POLYGON**_;
  
  7- _**GEOMETRYCOLLECTION**_: conjunto de geometrias de qualquer tipo das anteriores mencionadas.

Existem mais 10 geometrias especiais que não serão tratadas neste momento e que podem ser consultadas na documentação do pacote `sf`. 

O armazenamento das feições se efetua da seguinte maneira:

  + o **objeto `sf`**, classe 3, possui as seguintes propriedades: 
  
    1- um tipo geometria (alguma das mencionads anteriormente), 
    
    2- um tamanho (quantidade de feições e atributos), 
    
    3- uma extensão espacial ( _bounding box_ ou _bbox_), e um 
    
    4- _CRS_ (que pode ser desconhecido, ou seja  _NA_).

  + os atributos das feições são armazenados num objeto de tipo `dataframe`, denominado de classe`sf`, que contém obrigatoriamente as seguintes duas classes:
  
    1- objeto da classe `sfc`. Devido a que a geometria pode ter diversos valores, ela é armazenada numa única coluna contendo objetos tipo `list`. Essa coluna tem o mesmo tamanho do numero de registros (linhas) no `dataframe`;
  
    2- Objeto da classe `sfg`. A geometria de cada feição e armazenada num objeto `list`, contido dentro do objeto `sfc`.
  
A seguinte figura mostra como esta organizado um objeto desta classe `sf`. No cabeçãlho se indica a quantidade de feições e atributos, o tipo e dimensão da geometria, a extensão espacial e o CRS. Em amarelo destáca-se o dataframe (classe `sf`), em verde a geometria (numa coluna de tipo lista, classe `sfc`) e em vermelho a geometria de uma feição (neste caso do 3o. registro, classe `sfg`). 

O objeto da figura é um arquivo multipolígono contendo 6 feições com três variáveis cada. Os 6 registros correspondem ao limites de alguns municípios de RJ e as três variáveis são: um identificador, o geocódigo do município e o nome do mesmo.

 Cada classe mencionada tem seus próprios metodos, para ver a lista completa deles, execute `methods(class = "sf")`. Experimente ver os métodos das outras duas classes, `sfc` e `sfg`.

 
 </div>
  
 <div style="text-align:center" markdown="1">

![Classe de objetos na feição simples](sf.png)

</div>
 
---

<div style="text-align:justify" markdown="1">

#### c- Dados matriciais ou _raster_

Um arquivo ou dado _raster_ é uma estrutura de dados que divide o espaco em porções retangulares iguais chamadas células ou _pixels_, as quais podem armazenar um ou mais valores em cada célula. Essa estrutura é também conhecida como **grade** ou **quadricula**, em contraposição aos dados de tipo vetorial já vistos.

O pacote `raster` tem funções para ler, manipular e gravar dados matricias em forma muito eficiente, já que não precisa carregar os dados em memoria para executar as operacoes mencionadas. Isto se torna relevante já que normalmente os dados raster são de grande volume. Os objetos criados com esse pacote são da classe S-4.

O pacote `raster`permite importar tem 3 tipos de objetos. Básicamente todos eles armazenam os seguintes atrubutos ou _slots_:

    - o número de colunas e linhas, 
    
    - as coordenadas da extensão espacial (um objeto da classe "extent") e 
    
    - o SGR (objeto da classe "proj.4" além do código EPSG, se disponível). 

O _raster_ também pode armazenar o(s) valor(es) específicos de cada célula e variável (camada). **Importante: As coordenadas estão referidas aos extremos da grade (esquinas) e não ao centro de cada célula como o fazem alguns software de SIG.**

Os três tipos de objetos raster são: 

  + `RasterLayer`, representa um camada única de dados. Um exemplo pode ser um modelo digital de elevações (MDE) ou a superfície de temperaturas produzidas por uma rede de estações metereológicas;
  
  + `RasterBrick`, usado para representação de mais de uma variável ou camada ( _multi-layer_), mas que pertencem a um único arquivo. Um típico exemplo é uma imagem de satélite multiespectral; e 
  
  + `RasterStack`, para representação de mais de uma variável, as quais podem pertencer a diferentes arquivos ou a uma combinação de algumas camadas de um arquivo único. É obligatório que os limites espaciais e a resolução de todas as camadas sejam as mesmas.

A forma mais simples de criar um objeto _raster_ é através da leitura de um arquivo. O pacote permite ler os formatos mais comuns como GeoTIFF, ERDAS, ESRI, ENVI, etc, usado a biblioteca `rgdal`. Dependendo do tipo de arquivo que importamos serão usadas as funções `raster()`, `brick()` ou `stack()`.

</div>

---

<div style="text-align:justify" markdown="1">

### 3- Sistema Geodésico de Referência – SGR ( _CRS_)

Quando falamos em dados georreferenciados ou espaciais, estamos dizendo que eles tem um atributo que os relaciona com a superfície terrestre, esse atributo é a geometria como foi definida no título anterior. A relação com a superfície terrestre é estabelecida por meio de coordenadas, que pertencem a um **Sistema Geodésico de Referência – SGR**. No _R_, o SGR é chamado de **_Coordinate Reference System – CRS_**. 

Os elementos integrantes de um _CRS_ são: um **modelo da Terra**, um **sistema de coordenadas** e um **ponto terrestre de origem**, no qual se posiciona a origem do nosso sistema de coordenadas. 

Referente ao **modelo da Terra**, são usados principalmente dois modelos: o **esférico** e o **elipsoidico**. Este último tem uma maior aproximação à forma real da Terra, já que se definem dois eixos do elipsoide de diferentes tamanhos, e assim é possível modelar o achatamento existente nos polos (o raio polar é aproximadamente 11,5 km menor que o equatorial). Em algumas aplicações cartográficas não existe diferença em usar qualquer um desses modelos.

Os **sistemas de coordenadas** podem ser do tipo **geográfico** (definidos por latitude e longitude) ou **projetado** (eixos Leste e Norte, ou X e Y). A escolha determina como os eixos do modelo terrestre são divididos e as unidades de medida. Enquanto as unidades dos sistemas geográficos são angulares e medidas em graus, as dos sistemas projetados são lineares e medidas em metros (m).

**Ponto terrestre de origem**, esse ponto determina a origem da escala de nossos eixos de coordenadas. Anteriormente era definido na superfície terrestre por meio de medições astronômicas (sistema local). Nos dias de hoje, os sistemas de referência tem sua origem no geocentro, definido usando técnicas de posicionamento por satélites (sistemas globais).

Os elementos definitórios mencionados: **modelo terrestre, ponto terrestre de origem e , sistema de coordenadas**, caraterizam cada SGR. Em particular a escolha dos elementos: modelo terrrestre e o ponto terrestre de origem denomina-se de _Datum_. O _Datum_ SIRGAS2000, por exemplo, tem o elipsóide GRS80 como modelo terrestre e o geocentro terreste como ponto inicial. Nesse _Datum_ podemos escolher trabalhar com sistemas de coordenadas geográficas ou um sistema de coordenadas projetadas, como por exemplo o _Universal Transverse Mercator - UTM_ para a localização das feições.

<div/>

\
---
\

<div style="text-align:justify" markdown="1">

#### a- _CRS_ não projetado ou de coordenadas geográficas

Na definição do sistema, vimos que são escolhidos um elipsoide determinado (SIRGAS, WGS84, Internacional, etc.), um ponto de origem (modernamente o geocentro) e um sistema de coordenadas. Em relação as coordenadas do tipo geográficas, usa-se a **latitude** e a **longitude** para a localização espacial dos objetos. A **latitude** é a distância angular a partir do paralelo 0º, ou Equador, contada no sentido Polo Sul (latitude -90º) ou no sentido Polo Norte (latitude +90º). A **longitude** é a distância angular a partir do meridiano 0º, ou de Greenwich, contada no sentido leste (longitudes de 0º a 360) ou no sentido leste e oeste (longitudes de 0º a +180º e de 0º a -180º, respectivamente). Estas unidades de medida estão em graus sexagesimais. Na figura um sistema de coordenadas geográficas mostrando a localização de um ponto de coordenadas com longitude e latitude de -40 graus (Fonte: IBGE).

<div/>

<div style="text-align:center" markdown="1">

![SGR](sgr.png)


<div/>

---

#### b- _CRS_ de coordenadas projetadas

Os sistemas de referência com coordenadas planas ou projetadas supõem a projeção das coordenadas geográficas sobre uma superfície plana e o uso de eixos cartesianos X e Y, com escala métrica e origem determinada. Todos os sistemas projetados provem de algum sistema geográfico existente.

Para passar de um sistema de coordenadas geográficas para um de coordenadas projetadas, precisamos de uma superfície que tenha a possibilidade de se desenvolver no plano. Assim, temos projeções que usam superfícies cônicas, cilíndricas, ou planas. Para regiões localizadas em latitudes médias as superfícies cônicas com um ou dois meridianos de contato são as mais adequadas, enquanto para a cartografia global as projeções cilíndricas são melhores. Já nos polos, podemos usar uma superfície plana, tangente no próprio polo.

Os sistemas projetados tentam minimizar a deformação produzida na transformação. Desta forma, alguns preservam determinada geometria, como a área, a direção, as distâncias ou a própria forma dos objetos em detrimento dos outros.  **Não existe projeção que esteja livre de deformações, seja em direção, área ou forma das feições!** Na seguinte figura (Fonte: IBGE) se apresentam alguns dos sistemas de projeção mais usados.


<div style="text-align:center" markdown="1">

![Superificies CRS Projetados](proj.png)

<div/>

---

<div style="text-align:justify" markdown="1">

A biblioteca **_GDAL (Geospatial Data Abstraction Library)_** é uma biblioteca de licença aberta de software de computação para leitura e escrita de dados geoespaciais, seguindo a norma ISO mencionada na introdução.

A descripção de um _CRS_ geográfico se da através de um texto em formato `proj4string`, que define os parâmetros de cada sistema e é usada pela biblioteca `PROJ.4`.

Os sistemas de referência geodésicos projetados, são definidos através das especificações contidas no documento **_EPSG Geodetic Parameter dataset_** da **_International Association of Oil & Gas Producers - IOGP_** . Ambas bibliotecas estão implementadas no _R_ através da biblioteca `rgdal`. 

Nos arquivos vetoriais tipo camada ou _shape_ existem vários arquivos que definem a geometria e caraterísticas dos dados. Especificamente o arquivo `.prj` de cada `shape` contém as especificações do _CRS_. No _R_, existem diversas formas de definir um SGR, pode ser através de seu número EPSG, no formato `proj.4` ou no formato _Well Known Text - WKT_. Os seguintes exemplos estão no formato _WKT_, sendo que a letra em negrita mostra os argumentos de definição:

<div/>

---


##### _CRS_ não projetado ou de coordenadas geográficas


**GEOGCS**["SIRGAS 2000",

**DATUM** ["D_SIRGAS_2000", 

**SPHEROID**["GRS_1980", 6378137, 298.257222101]], 

**PRIMEM**["Greenwich",0], 

**UNIT**["Degree", 0.017453292519943295]]

---


##### _CRS_ com coordenadas projetadas


**PROJCS**["SIRGAS_2000_UTM_zone_24S", 

**GEOGCS**["GCS_SIRGAS 2000", 

**DATUM**["D_SIRGAS_2000", 

**SPHEROID**["GRS_1980",6378137,298.257222101]], 

**PRIMEM**["Greenwich",0],UNIT["Degree",0.017453292519943295]],

**PROJECTION**["Transverse_Mercator"], 

**PARAMETER**["latitude_of_origin", 0], 

**PARAMETER**["central_meridian", -39],

**PARAMETER**["scale_factor", 0.9996], 

**PARAMETER**["false_easting", 500000], 

**PARAMETER**["false_northing",10000000],

**UNIT**["Meter", 1]]

---

<div style="text-align:justify" markdown="1">

Para dados tipo raster, a mesma informação pode estar em arquivo separado ou incluído no cabeçalho do próprio arquivo, junto com os dados.

As seguintes tabelas resumem os principais SGR, com coordenadas geográficas e projetadas, que podemos encontrar no Brasil.

<div/>

---

#### Sistemas Geodésicos de Referência, coordenadas geográficas

---

<table class="table table-striped table-hover table-condensed" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:center;"> Nome </th>
   <th style="text-align:center;"> código EPSG </th>
   <th style="text-align:center;"> Definição formato Proj.4 </th>
   <th style="text-align:center;"> Observações </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> Córrego Alegre </td>
   <td style="text-align:center;"> 4225 </td>
   <td style="text-align:center;"> “+proj=longlat +ellps=intl +towgs84=-206,172,-6,0,0,0,0 +no_defs” </td>
   <td style="text-align:center;"> Obsoleto </td>
  </tr>
  <tr>
   <td style="text-align:center;"> SAD69 </td>
   <td style="text-align:center;"> 4291 </td>
   <td style="text-align:center;"> “+proj=longlat +ellps=GRS67 +towgs84=-57,1,-41,0,0,0,0 +no_defs” </td>
   <td style="text-align:center;"> Obsoleto </td>
  </tr>
  <tr>
   <td style="text-align:center;"> WGS84 </td>
   <td style="text-align:center;"> 4326 </td>
   <td style="text-align:center;"> “+proj=longlat +datum=WGS84 +no_defs” </td>
   <td style="text-align:center;"> Não oficial </td>
  </tr>
  <tr>
   <td style="text-align:center;"> SIRGAS2000 </td>
   <td style="text-align:center;"> 4674 </td>
   <td style="text-align:center;"> “+proj=longlat +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +no_defs” </td>
   <td style="text-align:center;"> Oficial </td>
  </tr>
</tbody>
</table>

---

#### Sistemas Geodésicos de Referência, _Datum_ SIRGAS2000 - coordenadas projetadas em sistema UTM

---

<table class="table table-striped table-hover table-condensed" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:center;"> Nome </th>
   <th style="text-align:center;"> código EPSG </th>
   <th style="text-align:center;"> Definição formato Proj.4 </th>
   <th style="text-align:center;"> Observações </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> UTM-18S </td>
   <td style="text-align:center;"> 31978 </td>
   <td style="text-align:center;"> “+proj=utm +zone=18 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 18 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-19S </td>
   <td style="text-align:center;"> 31979 </td>
   <td style="text-align:center;"> “+proj=utm +zone=19 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 19 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-20S </td>
   <td style="text-align:center;"> 31980 </td>
   <td style="text-align:center;"> “+proj=utm +zone=20 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 20 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-21S </td>
   <td style="text-align:center;"> 31981 </td>
   <td style="text-align:center;"> “+proj=utm +zone=21 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 21 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-22S </td>
   <td style="text-align:center;"> 31982 </td>
   <td style="text-align:center;"> “+proj=utm +zone=22 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 22 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-23S </td>
   <td style="text-align:center;"> 31983 </td>
   <td style="text-align:center;"> “+proj=utm +zone=23 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 23 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-24S </td>
   <td style="text-align:center;"> 31984 </td>
   <td style="text-align:center;"> “+proj=utm +zone=24 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 24 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-25S </td>
   <td style="text-align:center;"> 31985 </td>
   <td style="text-align:center;"> “+proj=utm +zone=25 +south +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 25 Sul </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-19N </td>
   <td style="text-align:center;"> 31973 </td>
   <td style="text-align:center;"> “+proj=utm +zone=19 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 19 Norte </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-20N </td>
   <td style="text-align:center;"> 31974 </td>
   <td style="text-align:center;"> “+proj=utm +zone=20 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 20 Norte </td>
  </tr>
  <tr>
   <td style="text-align:center;"> UTM-21N </td>
   <td style="text-align:center;"> 31975 </td>
   <td style="text-align:center;"> “+proj=utm +zone=21 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs” </td>
   <td style="text-align:center;"> Fuso 21 Norte </td>
  </tr>
</tbody>
</table>

---

<div style="text-align:justify" markdown="1">

Se precisar de um _CRS_ diferente o de outro pais, podemos consultar o site do [EPSG](epsg.io) e escolher aquele desejado. Ainda temos a possibilidade de exportar o _CRS_ em formato **WKT** ou **Proj.4**. Porém existe a possibilidade de um sistema de coordenadas projetadas não ter código EPSG, no caso de sistemas criados localmente e não globais.

<div/>

#### Exercicio 01

Temos dados antigos do Mocambique em coordenadas projetadas e datum Tete. Procure na pagina do EPSG o CRS correspondente e responda as seguintes perguntas:

    - qual a unidade de medida?
  
    - qual o elipsoide usado?
  
    - qual a autoridade responsavel pelo _CRS_?
  
Exporte o CRS em formato **WTK** e **Proj.4**.

\
---
\

FIM primeira parte

---

CREDITOS: 

  + IBGE - Nocoes Basicas de Cartografia - Manuais Tecnicos em Geociencias, 1999.
  
  +  Edzer Pebesma (2018). sf: Simple Features for R. R package version 0.6-3.
  https://CRAN.R-project.org/package=sf

  +  Robert J. Hijmans (2017). raster: Geographic Data Analysis and Modeling. R package version
  2.6-7. https://CRAN.R-project.org/package=raster

  +   Roger Bivand, Tim Keitt and Barry Rowlingson (2018). rgdal: Bindings for the 'Geospatial' Data
  Abstraction Library. R package version 1.3-4. https://CRAN.R-project.org/package=rgdal
 
  + [EPSG](https://www.epsg-registry.org/) 
  
---
