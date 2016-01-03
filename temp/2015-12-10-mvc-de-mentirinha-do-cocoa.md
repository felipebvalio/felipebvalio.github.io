---
layout:     post
title:      O MVC de Mentirinha do Cocoa
date:       2015-12-10 17:47
summary:    Como implementar o padrão MVC de verdade
categories: jekyll
thumbnail:  cogs
tags:
 - mvc
 - cocoa
 - ios
 - conductor
---

Apesar da Apple afirmar que Cocoa segue o padrão MVC, isso não é inteiramente verdade. Há quem afirme que foi criada uma variação, M/VC, pois as camadas view e controller foram fundidos em uma só, o view-controller. Ao mesmo tempo, alguns defendem que esta variação é na verdade um MVVC (Model-View-ViewController), pois ainda existe a camada de visão, porém magra, enquanto o view-controller é um controller que assume parte das funções da view.

Independente de como nomeemos o padrão seguido por Cocoa, um aspecto é unânime: view-controllers são extremamente inchados. Se seguirmos a implementação de um UIViewController da forma tradicional, como as próprias documetações da Apple sugerem, teremos algo mais ou menos assim:

 ---------
| Visão   |
|=========|
| UI      | }
| Lógica  | }-ViewController
| Sistema | }
|=========|
| Modelo  |
 ---------

Ou seja, um controller que assume várias tarefas. Uma delas, a de lógica, é a única que ele deveria conter. No entanto, ele também possui código para manipular a interface, confgurando cores, tamanhos, animações etc. Por sistema, entenda como código dependente de hardware, como tipo de aparelho (iPhone, iPad), tamanho de tela ou versão do iOS.

Até aí, podemos resolver o problema de view-controllers grandes apenas com boas práticas de programação, como criar componentes ou delegar tarefas para classes menores. Mas isso não nos ajuda se desejamos escrever testes automatizados para um view-controller. Se, por exemplo, tivermos uma lógica específica para o iPhone e outra para o iPad, como escreveremos testes para cada um sem termos que executar os testes uma vez no iPhone e outra no iPad?
