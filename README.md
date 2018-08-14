#Diretriz para configuração de limits dos pods da sua aplicação
Para configurar sua os limits dos containers dentro do Pod você só precisa entender como funciona alguns conceitos dentro do Kubernetes

<p>CPU e memória são cada um, um tipo de recurso (resource type). Um recurso possui uma unidade de medida. CPU é medida em unidades de cores de um processador, e memória é medida em unidades de bytes</p>

#Como definir a configuração de CPU e memória dentro do kubernetes:
```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

```
---

#Diferença entre request e limits

Basicamente request seria uma reserva de recurso ("quanto de memória e/ou CPU eu preciso para rodar minimamente a app?") e limits, como o nome já diz, é o máximo de CPU e/ou memória que eu posso ceder para cada container dentro do POD. Lógicamente os valores de request nunca podem ser maiores que os valores de limits.

---

Agora que você já entendeu os conceitos de recursos, requests e limits precisamos descobrir como descobrir a quantidade de recursos consumidos pelo seu container<br>
Execute o comando

```docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"```

Você verá uma tela exibindo o nome do container, a quantidade de CPU e MEMÓRIA em utilização pelo container. A partir desses números você já consegue definir os seus <b>resource requests</b>. Já os limits dependem de um teste de carga.

---

#Teste de Carga
Para os testes de carga sugerimos a utilização do <a href=https://gettaurus.org>Taurus</a>. Uma framework de testes opensource criada pela Blazemeter. Para começar a usar é muito simples (para melhores resultados instale o bzt em uma máquina e suba seu container em outra máquina, assim você não terá disparidades de resultado na hora dos testes)

>pip install bzt

caso não funcionar siga as recomendações <a href=https://gettaurus.org/docs/Installation/>desse link</a>

inicie sua aplicação pelo docker e crie um arquivo de configuração chamado quick_test.yaml
```
execution:
- concurrency: 100 #quantos usuarios simultâneos você espera que acessem a sua aplicação
  ramp-up: 1m #em quanto tempo o número de usuários chegará ao máximo
  hold-for: 5m #tempo em que sua aplicação será acessada constantemente pelo número máximo de usuários
  scenario: quick-test #nome do scenario

scenarios:
  quick-test:
    requests:
    - http://<url-para-sua-app>:<porta>
```

depois execute: (o -report cria um report html que nos permite visualizar os resultados de uma forma mais clara)
> bzt quick_test.yml -report

O BZT irá simular um número de usuários interagindo com sua aplicação, após o termino dos testes você conseguirá visualizar os reports a partir de um link que será exibido no seu terminal. Caso você necessite de testes mais complexos (multiplas chamadas diferentes, teste de carga distribuido, teste com ssl, etc) não tenha medo de visitar a documentação <a href=https://gettaurus.org/docs/Index/>clique aqui</a> :D

Durante o teste de carga acompanhe a quantidade de recursos utilizados (usando o comando mencionado anteriormente para acompanhar o consumo de recursos do container) pela sua aplicação para setar os limits. Digamos que nos testes, durante o pico máximo de usuários, o container estava consumindo 2 GB de memória RAM e 80% de CPU. Para configurar os limits corretamente, pense em um número pelo menos 30% maior que os atingidos durante o teste de carga.  

O Taurus integra com o Jenkins Pipeline também. Você pode rodar seus testes de carga durante a pipeline paralelamente seguindo os passos <a href=https://jenkins.io/blog/2017/08/17/speaker-blog-blazemeter/>desse link</a>

---
Obs:

1º O kubernetes não limita Pod limita container. Os limites de um pod seriam a soma limits/requests de cada container no Pod

2º Ao chegar no limite imposto de CPU o POD não é marcado para morrer. Já com excesso de memória, o POD pode ser finalizado.

Para qualquer tipo de problema veja a sessão de <a href=https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#troubleshooting>Troubleshooting</a> da documentação do kubernetes  
