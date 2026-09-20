# 5. Refinamento pós-lançamento

Se depois do lançamento, 30% dos clientes abandonam na etapa de validação, eu faria o seguinte:

Antes de mudar qualquer coisa, eu descobriria onde está acontecendo e por quê, primeiro definiria uma métrica de abandono, que seria quem recebeu o pedido do documento e não concluiu a validação. Depois olharia para o funil, etapa por etapa: mensagens enviadas, respostas dadas para a abertura, pedidos de documento, documentos enviados, documentos validados, e faria uma análise de 30 a 50 conversas de quem abandonou, separando o motivo: número errado, desinteresse, desconfiança, erro de digitação, erro técnico ou silêncio, porque nem tudo isso é abandono de verdade, e só o que sobra é problema realmente.

## Hipóteses

1. **Pedir o CPF pelo WhatsApp parece golpe.** O sinal seria "é golpe?" e o cliente sumir logo depois do pedido, nesse caso eu ajustaria o pedido para dizer o motivo, lembrar que o banco nunca pede senha ou token, e oferecer o app como alternativa.
2. **Digitar o documento dá trabalho, e o erro gasta a tentativa.** O sinal seria muitos avisos de formato inválido seguidos de silêncio. O erro de formato deixaria de contar como tentativa, já que o backend confere o formato antes da Mia, eu também avaliaria pedir só uma parte do documento, como "digite os 4 últimos dígitos do seu CPF".
3. **A abertura com teaser cria um turno a mais.** O sinal seria a queda entre a abertura e o pedido, nesse caso eu testaria pedir o documento já na abertura com o motivo explícito.
4. **Horário e frequência dos lembretes.** O sinal seria o abandono concentrado em certos horários, ou clientes que respondem só depois do lembrete.
5. **Falha técnica contada como abandono.** O sinal estaria nos erros da tool nos logs, o backend faria de novo a chamada e alertaria a equipe.
6. **No CNPJ, quem responde não é o responsável.** O sinal seria a taxa de "não sou eu" só nos casos de CNPJ, dessa forma eu pediria um segundo fator ou o contato do responsável.

## Ajustes que eu faria

São mudanças rápidas e de baixo risco, como por exemplo medir o abandono por motivo, reescrever o pedido do documento com o motivo e os canais oficiais, não gastar tentativa com erro de formato e refazer a chamada da tool quando ela falhar.

Depois, eu dividiria os clientes e testaria três coisas, uma de cada vez: a abertura (com pergunta contra pedido direto do documento), o texto do pedido (com e sem a justificativa) e o documento (completo contra parcial). A métrica principal é a porcentagem que conclui a validação. Olharia também opt-out, reclamações e tentativas fraudulentas, e, se qualquer um desses piorar, o experimento para.

## O que eu não mudaria

O limite de tentativas, a mensagem genérica de falha e a regra de não falar de oferta antes da validação. Pedir só parte do documento reduz o atrito, mas também reduz o rigor da validação, então só entraria depois de aprovação de segurança.
