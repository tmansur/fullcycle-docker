# Desafio Go

## Descrição do desafio:

Você terá que publicar uma imagem no docker hub.

Quando executarmos: `docker run <seu-user>/fullcycle`

Temos que ter o seguinte resultado: **Full Cycle Rocks!!**

Se você perceber, essa imagem apenas realiza um print da mensagem como resultado final, logo, vale a pena dar uma conferida no próprio site da Go Lang para aprender como fazer um "olá mundo".

Lembrando que a Go Lang possui imagens oficiais prontas, vale a pena consultar o Docker Hub.

**A imagem de nosso projeto Go precisa ter menos de 2MB**

## Executando a solução:

`docker build -t tmansur/hello .`

`docker run tmansur/hello`
