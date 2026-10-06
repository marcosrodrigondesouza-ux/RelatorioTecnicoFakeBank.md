               Relatório Técnico: Exploração de Vulnerabilidade de Controle de Acesso (Broken Access Control)

Plataforma: TryHackMe

Alvo: FakeBank ([http://fakebank.thm](http://fakebank.thm))

Vetor de ataque: Navegação forçada (forced browsing) via força-bruta de diretórios
Ferramentas utilizadas: dirb (via terminal)


1. Introdução e visão executiva (Purple Team)
A segurança ofensiva consiste em pensar como um atacante para encontrar e mapear vulnerabilidades antes que agentes maliciosos o façam. Este laboratório demonstrou a exploração de uma vulnerabilidade crítica de controle de acesso (Broken Access Control) em uma aplicação bancária simulada.

Através de simulações de ataques reais, foi possível descobrir e acessar um painel administrativo oculto, permitindo a transferência não autorizada de fundos. A falha evidencia o perigo de basear a arquitetura de proteção de qualquer sistema crítico em "segurança por obscuridade".

2. Metodologia do ataque (Red Team)
O ataque seguiu o fluxo padrão de reconhecimento e exploração:

Fase de reconhecimento: a aplicação principal estava mapeada no domínio [http://fakebank.thm](http://fakebank.thm). Para identificar diretórios ocultos não listados no código-fonte ou na interface, foi executada a ferramenta de força-bruta de diretórios (dirb) através do terminal.

Vetor descoberto: a ferramenta analisou as rotas e retornou a existência de diretórios válidos na aplicação. Dois URLs ocultos foram encontrados: [http://fakebank.thm/images](http://fakebank.thm/images) e [http://fakebank.thm/bank-transfer](http://fakebank.thm/bank-transfer).

Exploração: o acesso direto à URL [http://fakebank.thm/bank-transfer](http://fakebank.thm/bank-transfer) contornou o fluxo normal da aplicação. A página exibia um painel administrativo de transferências sem a exigência de credenciais ou validação de sessão.

Impacto: através do painel exposto, foi executada uma transferência ilícita no valor de $80.000 para a conta do atacante, positivando o saldo e comprometendo a integridade da instituição financeira (Flag obtida: BANK HACKED).

3. Análise da vulnerabilidade (Root cause)
O erro principal que possibilitou o ataque foi a falha no controle de acesso.

Os desenvolvedores presumiram que, como não havia nenhum botão ou link na interface apontando para a página de transferências, ela estaria protegida contra usuários comuns. Não houve validação no servidor (backend) para verificar se o usuário requisitando aquela URL específica tinha privilégios administrativos para visualizar ou interagir com o painel.

4. Estratégias de mitigação (Blue Team)
Para corrigir a vulnerabilidade e impedir o ataque, a arquitetura da aplicação deve implementar as seguintes defesas:

Autenticação e sessões: a página de transferências jamais deve ser carregada sem que haja um token de sessão válido confirmando a identidade de quem está acessando.

Controle de acesso baseado em funções (RBAC): além de estar logado, o servidor deve validar a função do usuário. Apenas usuários com a permissão administrativa no banco de dados devem acessar o endpoint. Caso contrário, o servidor deve retornar um erro de acesso negado (403 Forbidden).

Bloqueio de enumeração: para evitar que ferramentas como o dirb descubram diretórios, deve-se implementar bloqueios de requisições excessivas (rate limiting) e utilizar um firewall de aplicação (WAF) para bloquear padrões de tráfego de dicionários de força-bruta.

5. Conclusão: Do laboratório para a realidade em HealthTech
Transpondo esse ataque para a realidade clínica: se um invasor utiliza ferramentas de varredura em um sistema web de gestão laboratorial (como o TASY) e descobre um endpoint oculto como /prontuarios ou /admin-laudos, o impacto transcende o prejuízo financeiro.

A manipulação de valores de referência de exames sem restrição de privilégios compromete a integridade do diagnóstico e a vida do paciente, além de caracterizar uma violação gravíssima da LGPD. A defesa robusta não é apenas esconder a porta, mas garantir que ela só abra para a credencial correta.
