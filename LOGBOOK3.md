# Trabalho realizado na Semana #3

### CVE-2022-26959 SQL Injection 

### Grupo L01G02

## Identificação

- A NorthStar Club Management é uma empresa de gestão dados de clubes e tinha uma vulnerabilidade de SQL-Injection.
- Os utilizadores de IOS tinham a sua página de login comprometida, através de um atraso indevido de elementos especiais com um comando em SQL.
- Com o botão direito do rato aparece uma opção "Send to Repeater" que atrasa a base de dados do site.
- Era extraído da DB o nome, morada e informação do cartão de crédito dos clientes.

## Catalogação

- A empresa Assura descobriu a vulnerabilidade a 3 de Agosto de 2022, com auxílio de uma ferramenta Burp Suite.
- Através da introdução do comando 'sleep' em SQL no método POST da página, era possível aceder à DB da empresa.
- Bug bounty: Não houve qualquer tipo de recompensa.
- CVSS Score: 10, afetando todos as componentes do CIA.

## Exploit

- SQL injection no método POST da página, era possível ter acesso à base de dados da empresa.
- Basta usar o sqlmap.py para processar a base de dados.
- Não é necessário nenhum tipo de pré-package para realizar o exploit.
- A exploração da vulnerabilidade daria acesso ao maior ativo da NorthStar: acesso aos dados pessoais dos clientes.

## Ataques

- Não há registos de ataques, mas eram possíveis de se realizar.
- Como forma de mitigação, a NorthStar Club Management removeu a página afetada pela vulnerabilidade.
- Foi recomendado aos clientes ativar uma Web Application Firewall para detetar os ataques e informarem de outras vulnerabilidades.
- Até a este momento esta vulnerabilidade não foi resolvida.
