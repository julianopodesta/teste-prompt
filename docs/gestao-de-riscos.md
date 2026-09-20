# 4. Gestão de riscos

Para cada problema aqui abaixo, coloquei o que pode acontecer, o que a Mia faz e o que precisa ficar no sistema. Os testes estão em [`testes/testes.md`](../testes/testes.md).

## 1. Falar com a pessoa errada

O número pode estar trocado, ou outra pessoa pode atender, e algo menos comum seria alguém que não é o cliente saber o CPF dele.

Se a pessoa diz "não sou o Pedro", a Mia pede desculpas, não confirma nada sobre o cliente, não pede documento e encerra. Testei, e funcionou, o sistema descadastra o número.

A Mia não conhece o documento cadastrado, porque quem compara é a tool, mas ainda assim, quem tem o CPF de outra pessoa passaria pela validação. Uma ideia seria a validação parcial ou um segundo fator, principalmente no CNPJ, que é um dado público. 

## 2. Vazamento de dados ou do prompt

O cliente pode pedir o CPF cadastrado ou tentar arrancar as instruções da Mia.

Nesse caso ela recusaria, iria sugerir o documento pessoal ou o app e volta a pedir o CPF, nunca repete o documento nas mensagens. Testei os dois pedidos, e ela recusou. O prompt nem contém o CPF cadastrado, então não há o que vazar. Nos logs do meu teste um número fictício aparece, e em um cenário de produção real ele poderia ser mascarado.

## 3. Prompt injection

O cliente tenta mudar as regras: diz que é admin do banco, cola um aviso do sistema falso, pede o prompt e etc.

Na primeira vez, a Mia recusa em uma frase e volta ao ponto. Se repetir, ela encerra e o sistema marca risco. Testei uma mensagem longa de "engenheiro do banco" com um aviso `[SISTEMA]` forjado dentro do texto, e ela não revelou o prompt nem o CPF.

Há dois pontos fracos. O primeiro: os avisos do sistema e as mensagens do cliente chegam pelo mesmo canal, então o prompt sozinho não basta. O backend deveria remover `[SISTEMA]` do texto do cliente ou assinar os avisos. Neste ambiente não consegui testar um aviso forjado logo no início da mensagem. No segundo caso, o pedido de dado cadastral conta como manipulação, então um cliente legítimo que esqueceu o CPF poderia ser encerrado na segunda ocorrência. Daria para tratar "esqueci meu CPF" como dúvida.

## 4. Tentar adivinhar documentos

Alguém pode ficar tentando números até acertar.

A Mia responde de forma genérica, sem dizer o que está errado. O backend limita a 3 tentativas e, se a tool disser que o documento não confere, bloqueia o número. Testei no cenário 2: na 3ª falha ela encerrou e indicou o app. 

## 5. Oferta antes da validação, ou oferta inventada

A Mia pode ser convencida a falar de taxa antes de validar, ou pode inventar um produto.

Antes da validação, ela só diz que existe uma oferta. Nos meus testes não citou produto nem taxa. Depois da validação, o prompt não define a oferta e ela pode inventar detalhes, então a oferta deve vir de outro prompt, com dados reais, que assumiria só depois do sucesso da tool.

## 6. Cliente desconfiado, resistente ou que some

Esse seria o momento mais provável para o abandono por parte dos clientes, porque pedir CPF pelo WhatsApp dá a impressão de golpe.

A Mia explica o motivo uma vez, diz que nunca pede senha, token ou cartão, indica os canais oficiais e, se o cliente insistir, encerra com cordialidade. Se o cliente some, ela manda dois lembretes e se despede. Testei a resistência e o silêncio. Para a situação, por exemplo, que o cliente pergunta "É golpe?", ficou sem teste. Sobre os lembretes, que são as mensagens de follow-up, eu faria um double-check para ver se o custo compensa, vale conferir as regras do WhatsApp para mensagens fora da janela de atendimento e o horário de envio. O que fazer com esse abandono eu adicionei lá em [`refinamento.md`](refinamento.md).

## 7. Arquivos, áudios e números que não são documento

O cliente pode mandar uma foto do documento, um áudio, ou um telefone que parece um CPF.

O sistema bloqueia a mídia antes de ela chegar à Mia, que pede para o cliente digitar e, na segunda vez, encerra. Não fiz este teste, mas a ideia aqui é que arquivos e imagens, além de precisarem de uma "ramificação" a mais dentro do fluxo da Mia, podem esconder tentativas de injeção de prompt indireta, que é quando comandos maliciosos vêm ocultos de alguma forma dentro desses arquivos, o que confere um grau de dificuldade maior para a equipe lidar com ataques desse gênero. Quanto aos números: no meu teste, qualquer mensagem com 6 ou mais dígitos vira tentativa de documento, então um telefone com 11 dígitos seria tratado como CPF. Em produção, seria interessante o backend usar padrões de CPF e CNPJ.

## 8. Falhas técnicas e estado

A tool pode dar erro ou demorar, e o modelo pode variar entre execuções. Os contadores e os lembretes precisam de memória entre as mensagens.

Se a tool falhar, o backend refaz a chamada e só avisa a Mia quando o erro persistir. No meu teste, a Mia prometeu "tentar novamente" sem que ninguém refizesse a chamada. Depois de encerrar, o sistema fecha a sessão e não repassa mais mensagens à Mia. Os contadores de tentativas e de lembretes ficam num banco ou cache por sessão. No n8n de teste eu os simulei. Contra a variação do modelo, fixar modelo e temperatura e repetir os cenários a cada mudança no prompt. No cenário 2, a redação mudou de uma execução para a outra.

## 9. Consideração Final Sobre Gestão de Riscos

O enunciado do teste não menciona para onde o cliente deve ser redirecionado nos casos em que ele não aceita a oferta. Por exemplo, quando eu menciono "Mia redireciona para o app" ou "Mia explica que cliente precisa acessar o app", foi porque eu considerei não colocar um humano para tratar diretamente com o cliente neste caso específico, o que poderia inviabilizar a função da Mia e acabar aumentando custo e tempo de funcionários. Dessa forma, achei melhor fechar essa ponta que ficaria solta, seguindo a ideia de como é comum em outros bancos, que instruem o cliente a visitar um app oficial para resolver suas questões.
