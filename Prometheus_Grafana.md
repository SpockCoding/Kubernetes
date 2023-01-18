#### Adicionando o Prometheus como fonte de dados em nosso grafana, para geração do dashboard

* No cluster com o comando $kubectl get services copiar o endereço do prometheus server

![image](https://user-images.githubusercontent.com/97816800/213313331-c7ddd112-6a50-46ec-a28a-0f74e8b8dcc1.png)

* Iremos alterar o endereço mas manteremos a porta :9090, ir em Configurações > datasource > add datasource > selecionar o core prometheus > adicionar o IP que copiamos no cluster > save & test. 


![image](https://user-images.githubusercontent.com/97816800/213313722-5fb64295-d77f-4837-9d15-c8f8d2a674f0.png)



![image](https://user-images.githubusercontent.com/97816800/213313556-4a48a0a6-589c-4262-a586-19fbd67a8e30.png)



![image](https://user-images.githubusercontent.com/97816800/213312010-a3e4b7f3-af02-470f-a4fd-993324aef316.png)
