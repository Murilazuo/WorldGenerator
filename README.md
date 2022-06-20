# WorldGenerator
Protótipo de um gerador de terreno procedural na Unreal Engine

***
# Criação de terreno e props 

# Procedural Mesh

A geração procedural do terreno e dos objetos é feito através do componente Procedural Mesh que pode ser anexado à um ator, com esse componente é possível criar e manipular as Mesh e vértices de um ator. O ator pode possuir mais de uma mesh, isso é denominada section e podem ser acessada por seu índice.

## Create Mesh Section

Cria ou reescreve uma seção de mesh em um ator, essa função possui os seguintes parâmetros:

- Target : referência do Procedural Mesh componente em que será aplicada a função.

- Section Index: inteiro que representa o índice da seção que deverá ser criada ou reescrevida.

- Vertices : é um array de vectors que representa a posição das vértices da mesh. Por exemplo, para fazer um quadrado pode se usar o array { [0,0,0] , [0,1,0] , [1,1,0] , [1,0,0] }.

- Triangle : é um array de inteiros que indica o caminho de vértices que deve ser seguido(formando as arestas) para formar os triângulos, os elementos desse array devem ser múltiplos de 3. Seguindo no exemplo do quadrado, o array dos triângulos pode ser { 0 , 1 , 2 , 3 , 2 , 1 }, os inteiros representam os índices das vértices que serão ligados, então a vértice 0 ([0,0,0]) se conecta com a 1 ([0,1,0]) e depois com a 2 ([1,1,0]) e por fim com a 0 novamente, formando um triângulo. O mesmo acontece com outro triângulo, 3, conectando-se com 2, conectando-se com 1, conectando-se com 3 novamente. A união desses dois triângulos forma uma mesh em forma de quadrado.

- Normals : array opcional de normal vector para cada vértice, se for preenchido deve conter o mesmo número de elementos do array Vertices. Normals podem ser calculadas automaticamente com a função Calculate Tangemts for Mesh.

- UV0 ... UV3 : array opcional de vector 2D que representa as coordenadas de textura para cada vértice, deve ter o mesmo número de elementos do array Vertices.

- Vertex Color : array opcional de cores lineares que representa a cor de cada vértice, deve conter o mesmo número de elementos do array Vertices. 

- Tangents : array opcional da estrutura Proc Mesh Tangent que indica a tangente de cada vértice, deve ter o mesmo número de elementos do array Vertices. Assim como o Normals, a Tangents pode ser calculada automaticamente pela função Calculate Tangemts for Mesh .

- Create Collision: bool que indicará se a mesh section deverá ter colisão, essa opção adiciona um considerável custo em processamento.

# Structs

- NoiseMap: possui apenas uma variável, um array de float chamado Y. com um array dessa struct é possível uma espécie de array 2D.

# BP_Terrain

Classe responsável por gerar e desenhar o terreno e a água.
## Variáveis:

- NoiseMap RandomNoise: números aleatórios que servirá de base para fazer as vértices do terreno.
- NoiseMap NoiseMap: posições das vértices já suavizados.
- float WaterHeight: posição z do plano da água.

### Configurações de terreno

- int SizeX, SizeY: Largura e comprimento do terreno, em número de vértices.
- int Height: altura máxima que será gerada antes de suavizar. 
- float MinHeight, MaxHeigth: valor mínimo e máximo da altura, será utilizado na hora de suavizar o random noise.
- float DistanceToVertex: distância entre as vértices, o lado de cada quadrado do grid.
- float WaterHeghPercentage: porcentagem, em relação à altura em que a água será desenhada.

### Dados da Mesh
- Vertex
- Triangles
- UV
- Normals
- Tangents

## Create Terrain

Função responsável por definirgerar os dados do terreno (Generate Noise, Smoth NOise e Create Mesh Data), definir o material do terreno, criar a malha em sí (Update Mesh) e criar a malha da água(Create Water)

![image](https://user-images.githubusercontent.com/78811958/165130105-bb3aba7a-a2e9-47e0-846f-be8fa57bc211.png)

## Create Noise

array de float Y como variável local

Limpa os dados do Random Noise, for loop de 0 até Size X, no seu corpo, outro for loop de 0 até size Y, no seu corpo, adiciona um numero aleatório entre 0 e o valor de Height, ao final desse segundo loop, é adicionado o Y ao Random Noise e em seguida limpado os dados.

![image](https://user-images.githubusercontent.com/78811958/165138983-15a00ebb-5956-4fc8-bf82-46d0c070c3e8.png)

## Get Random Noise Position

Função responsável por retornar uma referencia de um elemento de Random Noise

![image](https://user-images.githubusercontent.com/78811958/165166884-9f36e0ba-3340-4ef3-805d-ecf42b7e551c.png)

## Smoth Noise

### Variáveis Locais

- indexX
- indexY
- Y

Função responsável por suavizar os dados de Random Noise, fazendo a média dos valores adjacentes.

Limpa os dados de NoiseMap, for loop de 0 até Size X, no seu corpo, outro for loop de 0 até size Y.

![image](https://user-images.githubusercontent.com/78811958/165163028-10307f02-c2f3-4c3b-9747-84447c5e890e.png)

No corpo do segundo for, por meio da função **Get Random Noise Position** utilizando como x, o index do primeiro for, e como y o do segundo, é somado todos os valores adjacentes e dividindo por 9 para fazer a média dos valores adjacentes. 

(pode ser substituído por um for dentro de outro for que vai de -1 a 1, sendo o index de um for a cordenada x e outro a cordenada y do noise map)
![image](https://user-images.githubusercontent.com/78811958/165167163-f2981b6e-eb89-43a9-8758-040fc982f4c8.png)


![image](https://user-images.githubusercontent.com/78811958/165165300-f894fb7b-fdfe-404c-a72a-96049efda883.png).

A média desses valores será adicionada a Y. Cada vez que o segundo loop for completo, Y será adicionado ao NoiseMap e logo depois será limpado.

![image](https://user-images.githubusercontent.com/78811958/165169289-a0f29e82-a4a6-481c-8b50-ff11c5c5b832.png)

## Get Noise Position

Função responsável por obter um elemento de Noise Map.

![image](https://user-images.githubusercontent.com/78811958/165170141-42548cf3-4006-44b2-8c02-17dc40ba9030.png)

### Create Mesh Data

Função responsável por preencher as variáveis de Dados da Mesh.

### Variáveis locais
- X
- V

### Limpar as variáveis

![image](https://user-images.githubusercontent.com/78811958/165170975-bbb4aee0-9571-4371-a1a4-51227f4740c3.png)

### Criação de Vertex e UV

Como já feito anteriormente, é feito um for dentro de outro, o index de cada loop será utilizado para definir X e Y.

![image](https://user-images.githubusercontent.com/78811958/165171924-d18b5d86-e364-4735-87ee-13a7575decf0.png)

É adicionado um elemento ao array vertex, sendo x = X * Distance to Vertex, y = Y * Distance to Vertex e z = valor de retorno de Get Noise Position utilizando como entrada X e Y.

![image](https://user-images.githubusercontent.com/78811958/165172695-6336cf89-2ad0-41a8-bca6-d64c0e46f747.png)

Em seguida é adicionado o valor de X e Y ao array UV.

![image](https://user-images.githubusercontent.com/78811958/165172968-7a0a5ec4-4d52-4bed-a3df-cec5e31f6e1f.png)

### Criação de Triangle, Tangents e Normals

Para os triângulos, é utilizado uma função proveniente do componente ProceduralMesh, a Create Grid Triangles, que retornará um array para os Triangles recebendo os valores de SizeX e SizeY.
Em seguida é utilizado a função, novamente proveniente do ProceduralMesh, Calculate Tangents for Mesh, que recebe os array de Vertex, Triangle e UV para retornar um array de Normals e Tangents.

![image](https://user-images.githubusercontent.com/78811958/165174724-21e6128b-9217-46db-9c9a-3976814baf85.png)

## Update Mesh

Cria uma malha com todos os array de dados da Mesh com a função Create Mesh Section do componente ProceduralMesh.

![image](https://user-images.githubusercontent.com/78811958/165175368-76270ced-f495-47a0-8c84-6606965bc5c5.png)

## Create water

Função responsável por criar um plano de água.

Define a variavel Water Height pra Height vezes Water Height Percentage.

![image](https://user-images.githubusercontent.com/78811958/165175922-23a75c5c-6b32-4901-915d-8754af465f67.png)

O Array de Vertex terá 4 elementos um para cada extremo do terreno, sendo um dos extremos 0 e outro SizeX e SizeY * Distance to Vertex, z será igual ao Water Height.

![image](https://user-images.githubusercontent.com/78811958/165176388-55684c6b-a3d1-4151-86c3-a503fd747e51.png)

O Array Triangles será preenchido manualmente, formando dois triângulos.

![image](https://user-images.githubusercontent.com/78811958/165176799-21d43e3e-0c55-453f-b701-2f41b31f5b4e.png)

O Array de UV será semelhante ao de Vertex, porém sem o z. 

![image](https://user-images.githubusercontent.com/78811958/165176980-32d00320-2ea2-4ca1-a82d-021a4a7bfc4d.png)

Por fim, é utilizado o Create Mesh Section, com Section Index = 1, diferente do index do terreno e depois defini o material da Section.

![image](https://user-images.githubusercontent.com/78811958/165177332-e8f437a4-a1e1-487a-8260-eb78336de3a6.png)

O Resultado Será semelhante a esse:

![image](https://user-images.githubusercontent.com/78811958/165177649-f7f0720b-1be4-4723-a9c6-f5cb93779b56.png)

## Add Random Noise Position

Incremente ou decrementa um valor em uma coordenada específica no Random Noise.

![image](https://user-images.githubusercontent.com/78811958/165178528-76e13297-e953-416a-82c0-95a23a6fab93.png)

## Change Terrain

Função responsável por modificar o terreno.

### Variáveis
- float To Add Z: valor a ser adicionado na coordenada específica.
- Vector Coord Terrain
- bool increment

![image](https://user-images.githubusercontent.com/78811958/165595762-53579af1-be47-4fb3-a942-71a810f1a806.png)

Atualiza a mesh

![image](https://user-images.githubusercontent.com/78811958/165179439-d38db4cb-c806-4630-9f35-25c9e7ecba5d.png)

# BP_SpawnObject

classe responsável por criar os objetos do mundo.

## Variáveis

- Density: número de objetos que tentará ser criado a cada 100 unidades do mundo.
- Scale Size: tamanho mínimo e máximo de cada objeto.
- Ref_Terrain: referencia ao ator do terreno, BP_Terrain.
- ToSpawn: referencia da classe que deverá ser spawnada.

## Generate Spawn Transform

Função responsável por gerar aleatoriamente a posição do ator em relação ao SizeX e SizeY do Ref_Terrain, além de definir um tamanho aleatório.

![image](https://user-images.githubusercontent.com/78811958/165595964-77b74bb1-c04e-4fd4-aa23-18aeee2f364c.png)

## SpawnObject

Função responsável por spawnar um objeto, utilizando a scale e location retornadas de Generate Spawn Transform.

![image](https://user-images.githubusercontent.com/78811958/165596209-b7d22c58-0934-499c-9266-cf07909bac1f.png)

## SpawnAllObjects

Função responsável por repetir a função SpawnObject um numero de vezes igual SizeX  *  SizeY / 100 (dividir todo o terreno em quadrantes) * Density. É chamado no começo do jogo.

![image](https://user-images.githubusercontent.com/78811958/165596334-5f7fcf73-b66f-45d0-854d-7706b85afe03.png)

## Reset Level

Destrói todos os atores com a classe ToSpawn e chama a função SpanwAllObjects.

![image](https://user-images.githubusercontent.com/78811958/165596527-e89d07f1-4679-4be0-9432-e7f5962d8111.png)

# Component_PutInGround

Componente que serve para retornar a posição que o objeto deve ser spawnado para ficar logo a cima do chão.

## Put In Ground

Recebe um referencia do ator que usará o componente. Faz um LineTracerByChannel (Função da Unreal que cria uma raio que detecta colisões em um canal específico) com começo na localização X e Y do objeto e Z igual a qualquer numero acima do ponto mais alto do terreno, e termino na localização X e Y do objeto e Z igual a um numero abaixo do ponto mais baixo do terreno. 

![image](https://user-images.githubusercontent.com/78811958/165596825-e0103c54-9dd4-4265-b178-43646d558cfa.png)

O line tracer retorna uma bool que indica se a linha colidiu ou não e um out hit, hit é uma estrutura que contém todos os dados dessa Line. Para pegar os dados de hit, devemos quebrar o resultado de ocorrência para pegar a posição do ponto de impacto e o ator que foi colidido. Se o z do ponto de impacto for menor que a altura da água, ou o a classe do componente colididos for **BP_Rock** ou **BP_Tree**, a função retornará a bool destroy como true, se não, retornará o ponto de impacto menos um valor suficiente para que o objeto fique completamente dentro do terreno.

![image](https://user-images.githubusercontent.com/78811958/165597029-575f052c-af79-45f1-ab6f-6dc765336f6a.png)

# BP_Tree e BP_Rock

Possuem comportamentos idênticos, mudando apenas sua colisão e seu render. Serão as classes utilizadas no BP_SpawnObject

## Put Rock in Ground / Put Tree In Ground

Função responsável por teleportar o ator para o chão utilizando o Component_PutInGround. é chamano no Begin Play.

Teleporta o ator para muito alto, Chama a função PutInGround, se a função retornar destroy como verdadeiro, o objeto será destruído, se não, ele será teleportado para a localização que PutInGround retornar.

![image](https://user-images.githubusercontent.com/78811958/165597469-033636eb-7b2d-491a-8db6-9640d3550dc5.png)

# ThirdPersonCharacter

Possui a movimentação básica da classe do projeto de terceira pessoa da Unreal, porém com algumas modificações. A modificação é a capacidade de manipular o terreno por meio da função **Change Terrain** do **BP_Terrain**. É necessário adicionar uma static mesh para servir como ponto de modificação da mesh, preferencialmente com um material translúcido.

![image](https://user-images.githubusercontent.com/78811958/165598833-da0db3bb-41f3-4a34-b3c1-a0226d068951.png)

Para o jogador manipular o terreno, é necessário ter um input de incremento e outro de decremento.

![image](https://user-images.githubusercontent.com/78811958/165598309-ae3dc90e-5c58-494b-9536-a9fadc148235.png)

***

O texto acima explica a primeira entraga desse trabalho, apenas com a criação do terreno e a movimentação básica do personagem. Na segunda parte, foquei em adicionar elementos de gameplay baseado no terreno e o spanw de props.

A ideia do protótipo é um jogo de sobrevivência, onde você precisa comer regularmente, se não o personagem morre. O jogador também pode coletar recurços, madeira, pedra e metal (além da comida) para fabricar outras ferramentas. O objetivo do jogo é sobreviver o maior tempo possível em partidas rápidas de alguns minutos.



# World Generator Survivor

## Component_SpawnProp
Esse componente nada mais é que o refatoramento do Component_PutInGround. Agora, o componente não só colocar o ator no terreno, como também gera sua posição aleatoriamente, então, ao invéz de destruir o objeto caso não possa colocar no chão, é gerado uma nova posição até que o objeto ache uma posição apropriada.

Geração da posição aleatória.
![image](https://user-images.githubusercontent.com/78811958/173621505-f2585d2b-bcf8-41ca-bb9f-dd17c91943be.png)

Definição de variáveis de referencias (passos 0,1 e 2) e loop para procurar uma posição viável.
![image](https://user-images.githubusercontent.com/78811958/173621916-a7617e0d-d487-487c-9202-8fca8f3f292e.png)

O objeto e posicionado em uma posição aleatória.
![image](https://user-images.githubusercontent.com/78811958/173622142-547684dd-4dda-47eb-b42b-c87a3547ff87.png)

É feito um line tracer em uma posição logo abaixo do objeto até a posição mais baixa do terreno.
![image](https://user-images.githubusercontent.com/78811958/173622496-88126fca-6559-4bbb-a16e-6bb47f44eaac.png)

Se a posição do impacto for menor que a posição da água ou o ator colidido não for o BP_Terrain, a função retorna destroy como true, se não retorna com false. Se destroy for igual a true, a função é chamada novamente.
![image](https://user-images.githubusercontent.com/78811958/173622834-d6f96cca-00c0-4e1b-b6a0-ba95167e2d1e.png)

## RandomRotate
Componente que adiciona uma rotação aleatória no eixo Z.

![image](https://user-images.githubusercontent.com/78811958/173623199-6f1063c1-7e1f-47ec-9bb0-8238ddada838.png)

## Component_CollectableObject

### Begin Play
A variável life é definida de acordo com a scale.
![image](https://user-images.githubusercontent.com/78811958/173623592-32f3c027-e1ec-409f-93f0-076837825fee.png)

### TakeDamage
Função que é chamda para dar dano no prop.

Caso o dano seja igual a -1, o player possui uma ferramenta que não coleta o prop, se for 0, ele só pode coletar props com uma scale menor.
![image](https://user-images.githubusercontent.com/78811958/173624558-0c3d5534-5cb5-4ff9-a7fc-5d41af8bb8ca.png)

Se for qualquer outro dano, a life do objeto é diminuida de acordo com o dano, se chegar a zero, o objeto é destruido.

![image](https://user-images.githubusercontent.com/78811958/173625353-5430b89d-3c73-40f0-9822-481c057b2145.png)

## Ferramentas e inventário

Para o inventário utilizei uma struct com cada recurço
![image](https://user-images.githubusercontent.com/78811958/173625615-21566862-84e0-4ee1-a395-79dc0c674aa1.png)

Para as ferramentas utilizei outra estrutura, a toolData, que contém o poder de coleta, o custo de fabricação, a mesh e o material da ferramenta.

![image](https://user-images.githubusercontent.com/78811958/173625770-e51e0848-4674-4a85-9448-1be1ae3c45a2.png)

Dentro do Blueprint do player há um array dessa estrutura, e pela função SetTool, é possivel alternar entre essas ferramentas.


### SetTool
Função do player que alera o render e o poder de coleta do player.
A função recebe um index para acessar os dados de toolData, retira os recursos necessários para a fabricação do item (a verificação da quantidade de recursos é feita no menu de craft).

![image](https://user-images.githubusercontent.com/78811958/173626467-d735372a-3b0e-4bcd-997b-babc25b4bc0c.png)

Logo em seguida, a variável de poder de coleta do player é atualizada (toolDamage), a malha e o material é alterado de acordo com o array toolData.
![image](https://user-images.githubusercontent.com/78811958/173627043-d1f786f4-782c-4f52-88d9-50b5cc0aab94.png)




