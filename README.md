# Robot-Project

## **DESENVOLVIMENTO E CONTROLE DE UM ROBÔ SEGUIDOR DE LINHA PARA USO INDUSTRIAL**

Trabalho apresentado como requisito parcial para a obtenção de nota na disciplina de Linguagens formais e autômatos, sob orientação do Prof. Eduardo.

### **RESUMO**

Este artigo apresenta o desenvolvimento de um robô seguidor de linha para uso industrial. O projeto aplica conceitos de gramática, alfabeto, autômatos finitos determinísticos para construir um protótipo capaz de seguir trajetórias pré-definidas. Utilizando sensores infravermelhos, motores DC , o robô foi testado em diferentes trajetos curvos e retos. Os resultados demonstraram a viabilidade do sistema com precisão satisfatória e respostas rápidas aos circuitos.

Palavras-chave: Arduino, Robô Seguidor de Linha, Sensores Infravermelhos

### **1 INTRODUÇÃO**

Este artigo científico apresenta o desenvolvimento de um robô seguidor de linha com aplicação industrial. Trata-se de um projeto para fornecer uma solução automatizada para o transporte de materiais, seguindo trajetos delimitados por faixas no chão.

### **2 FUNDAMENTAÇÃO TEÓRICA**

Robôs seguidores de linha são sistemas autônomos que detectam e seguem uma trilha marcada no chão, geralmente com cor contrastante (como por exemplo, preto no branco). O controle é geralmente feito com sensores IR que detectam variações de cor, enviando sinais a um microcontrolador que ajusta a velocidade dos motores.

O Arduino é bastante utilizado no desenvolvimento de sistemas semelhantes. Com sua linguagem simples e sua grande comunidade, facilita no desenvolvimento de projetos como este.

### **3 METODOLOGIA**

Componentes Utilizados:

- Arduino UNO R3
- 2 Sensores IR TCRT5000
- 2 Motores DC com caixa de redução
- Ponte H Dupla L298N
- Chassi com rodas e suporte para bateria
- Fonte de alimentação 7,4V (bateria Li-ion)

### 3.1 Diagrama do Circuito

O diagrama esquemático das conexões elétricas do protótipo é apresentado na Figura X 

![Diagrama]()

**Figura X:** Diagrama esquemático do robô seguidor de linha.

O diagrama ilustra a interconexão entre o microcontrolador Arduino UNO R3, a ponte H L298N, os dois motores DC e os dois sensores infravermelhos TCRT5000. O Arduino atua como a unidade central de processamento, recebendo os sinais dos sensores e comandando a ponte H, que por sua vez é responsável pelo acionamento e controle de direção dos motores. A alimentação do sistema de potência (motores) é fornecida pela bateria de Li-ion, enquanto o Arduino pode ser alimentado via USB ou pela mesma bateria através de um regulador (se aplicável, ou especificar se a L298N provê 5V para o Arduino). Esta configuração física é a base para a implementação da lógica de controle derivada do autômato finito.

### 3.2 Implementação do Código e Lógica de Controle baseada em Autômato

O comportamento do robô seguidor de linha, modelado conceitualmente como um Autômato Finito Determinístico (AFD), foi implementado em linguagem C/C++ para a plataforma Arduino. O código-fonte define as interações com os sensores e atuadores, traduzindo as transições de estado do autômato em comandos para os motores.

O alfabeto de entrada Σ do nosso autômato é formado pelas possíveis leituras combinadas dos dois sensores infravermelhos (S1 e S2). Considerando que cada sensor retorna um valor digital (0 para superfície branca e 1 para superfície preta, conforme explicitado no código), temos os seguintes símbolos de entrada:

- (0,0): Ambos os sensores sobre superfície branca (robô possivelmente centralizado ou perdendo a linha por completo se a linha for mais fina que a distância entre sensores).
- (1,0): Sensor S1 sobre preto (linha) e Sensor S2 sobre branco (robô desviando para a direita, necessita corrigir para a esquerda).
- (0,1): Sensor S1 sobre branco e Sensor S2 sobre preto (linha) (robô desviando para a esquerda, necessita corrigir para a direita).
- (1,1): Ambos os sensores sobre superfície preta (robô centralizado sobre a linha). *Nota: o código fornecido trata (0,0) como "em linha reta", o que sugere que a linha é preta e o fundo é branco, e os sensores estão calibrados para detectar a linha preta. A lógica `if((Sensor1 == 0) && (Sensor2 == 0))` para seguir reto implica que 0 é a linha. Se 0 é branco, então (0,0) significa que está fora da linha por ambos os lados ou a linha terminou. É crucial alinhar a descrição com o código.*

**Atenção:** No seu código, `// Para a cor branca atribuímos o valor 0 e, para a cor preta, o valor 1.`
Isso significa:

- `(0,0)` = (Branco, Branco) -> Robô está com ambos os sensores fora da linha preta, sobre o fundo branco.
- `(1,0)` = (Preto, Branco) -> Sensor esquerdo na linha preta, direito no fundo branco.
- `(0,1)` = (Branco, Preto) -> Sensor esquerdo no fundo branco, direito na linha preta.
- `(1,1)` = (Preto, Preto) -> Se a linha for grossa o suficiente. O código não trata esse caso explicitamente, ele seria pego pelo primeiro `if` se os sensores estivessem ambos em "não-preto".

Vamos reinterpretar as condições do código:
"Se detectar na extremidade das faixas duas cores brancas" -> `(Sensor1 == 0) && (Sensor2 == 0)` -> Segue em frente. Isso sugere que o robô deve seguir uma *ausência* de linha em ambos os sensores, ou seja, estar *entre* duas linhas ou seguir um caminho branco largo. Ou, mais provavelmente, que a lógica padrão é seguir em frente e as correções são feitas quando um sensor detecta a linha preta.

Para um seguidor de *linha preta em fundo branco* onde 0 é branco e 1 é preto:

- **Estado "Em Linha/Seguindo Reto"**: Idealmente, seria quando a linha está entre os sensores ou um padrão específico. No código, `(Sensor1 == 0) && (Sensor2 == 0)` (Branco, Branco) faz o robô ir para frente. Isso pode ser interpretado como "nenhum sensor detecta a linha, então continue reto" (assumindo que está no caminho certo).
- **Estado "Desvio à Esquerda Detectado" (Corrigir para Direita)**: `(Sensor1 == 0) && (Sensor2 == 1)` (Branco, Preto). O sensor direito (S2) detecta a linha preta. O robô vira para a direita (Motor M1 ligado, Motor M2 desligado).
- **Estado "Desvio à Direita Detectado" (Corrigir para Esquerda)**: `(Sensor1 == 1) && (Sensor2 == 0)` (Preto, Branco). O sensor esquerdo (S1) detecta a linha preta. O robô vira para a esquerda (Motor M1 desligado, Motor M2 ligado).

### **4 RESULTADOS E DISCUSSÕES**

Os testes foram realizados em trajetos curvos e retos. O robô teve um bom desempenho, com resposta rápida às mudanças de direção. Um ponto observado foi a interferência da luz ambiente podendo afetar a leitura dos sensores IR.

Tabela de desempenho:

- Linha reta: 100%
- Curvas leves: 90%
- Curvas acentuadas: 85%

### **5 CONCLUSÃO**

O projeto atingiu seus objetivos ao desenvolver um robô seguidor de linha funcional. O uso do Arduino e de componentes simples mostrou-se eficaz para um protótipo a pequena escala. Você pode acessar o resultado final indo em imagens neste mesmo repositorio

### Referencias

[https://blog.eletrogate.com/robo-seguidor-de-linha-tutorial-completo/](https://blog.eletrogate.com/robo-seguidor-de-linha-tutorial-completo/)
