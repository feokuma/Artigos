# 🧩 Como o OpenTelemetry ajuda times de desenvolvimento e operação a falarem a mesma língua

Por um grande período, times de desenvolvimento e times de operações viveram sem mundos separados quando o assunto é monitoramento de aplicações em produção.
Pessoas desenvolvedoras focadas em entregar features e corrigir bugs enquanto o operações olhando para painéis de controle e monitorando os feedbacks das pessoas usuárias para entender se tudo está funcionando como esperado. E no meio desse caminho... um grande ponto cego.

Quando algum problema acontece em produção, a pergunta é sempre a mesma: **“onde está o problema?”**  
Código? Banco de dados? Infraestrutura? Outro serviço/microsserviço?
E a resposta, quase sempre, é: **“depende de quem você pergunta.”**

## 👀 A falta de visibilidade cria silos

Este caso é muito comum, principalmente em empresas onde temos vários times, cada um olhando apenas para o seu próprio pedaço no todo — logs de aplicação, métricas de infraestrutura, traces de APIs, alertas do monitoramento... Cada time crie sua própria forma de ter um visão do que está acontecendo em produção e quando é necessário analisar um problema que se extende além dessas fronteiras, conectar esses pontos vira um exercício de adivinhação.

## 🔭 O OpenTelemetry como tradutor universal

É em cenários como o que descrevi acima que o OpenTelemetry muda esse jogo. Cria **um padrão aberto e comum para coletar dados de observabilidade** — logs, métricas e traces.  
Isso significa que o mesmo evento pode ser observado com o mesmo contexto, seja pelo desenvolvedor debugando localmente, seja pelo time de operações analisando um incidente em produção.

Com ele, todos falam a mesma língua porque estabelece um padrão para coletar, formatar e transmitir essas informações e quando todos falam a mesma língua, o foco volta a ser resolver o problema, não discutir de quem é a culpa.

## 🤝 Observabilidade como colaboração

Adotar OpenTelemetry não é só uma decisão técnica — é uma mudança cultural. Considerando que cada time é responsável por uma parte de um todo, quando estamos falando de uma grande solução com vários pequenos sistemas conectados, falar a mesma língua quando estamos gerando essas informações de Observabilidade permite que times compartilhem dados e percepções, criem dashboards comuns e, principalmente, **tomem decisões com base em fatos**. E isso tudo conseguindo entender não somente o que está acontecendo no que sua responsabilidade, mas também o que está acontecendo nos outros componentes com os quais o seu serviço está interagindo.  
O resultado é um ciclo mais saudável: menos retrabalho, menos ruído e mais aprendizado.

## 🚀 Um passo de cada vez

Parece muito legal né?! Mas calma, não precisa reescrever tudo. Este processo pode começar aos poucos instrumentando um serviço crítico, por exemplo, ou adicionando apenas traces em pontos de integração. Com o tempo, essa visibilidade vira parte natural do fluxo de desenvolvimento, e naturalmente, quando os times começam a perceber o poder que ganham ao conseguir saber o que está acontecendo na solução que está rodando em produção, a tendência é todos começam a sentir a necessidade de mais informações para entender comportamentos, momentos de pico, anomalias, e isso vai naturalmente criando nos times a cultura de instrumentar e enriquecer as informações que são coletadas, assim como melhorar a formatação e organização desses dados.

---

No fim das contas, o OpenTelemetry não é apenas sobre _ver_ o sistema, é sobre **entender** o sistema, como um todo incluíndo também partes deste todo que não somos responsáveis pelo seu desenvolvimento ou sustentação, mas que fazem parte da entrega de valor com o qual estamos contribuindo. Isso fortalece não só a qualidade do sistema, mas cria um cultura de colaboração entre times e pessoas. 👊

## Referências

- [OpenTelemetry](https://opentelemetry.io/)
- [CNCF](https://www.cncf.io/)
- [Doses de Telemetria](https://www.youtube.com/@DoseDeTelemetria)
- [marilya.dev](https://www.marylia.dev/)
