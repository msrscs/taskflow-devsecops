# TaskFlow (App de Exemplo) — Disciplina DevSecOps

Esta é a aplicação de laboratório usada em **todos os encontros práticos** da
disciplina. É um gerenciador de tarefas (to-do list) simples em Python/Flask,
propositalmente vulnerável, que evolui ao longo do curso:

- Módulos 1–2: usada para discutir arquitetura, pipeline e hardening.
- Módulo 3: alvo de SAST (Semgrep) e SCA (pip-audit / Trivy) — os alunos
  encontram SQLi, XSS, segredo hardcoded e dependências vulneráveis.
- Módulo 4: alvo de DAST (OWASP ZAP) rodando contra a aplicação em execução.
- Módulo 5: containerizada com Docker e provisionada via Docker Compose,
  usada para praticar scanning de IaC.
- Módulo 6: instrumentada com logging estruturado para observabilidade.
- Módulo 7: base do projeto final — os alunos entregam a versão corrigida
  com a esteira DevSecOps completa.

## ⚠️ Aviso importante

Este código contém vulnerabilidades **intencionais** para fins didáticos
(ver comentários `# FALHA` e docstring no topo de `app.py`). Nunca:

- Use este código como referência de boas práticas.
- Implante esta aplicação em ambiente de produção ou exposto à internet.
- Reutilize o padrão de código (concatenação de SQL, senhas em texto puro,
  segredo hardcoded) em projetos reais.

## Como executar localmente

```bash
cd app-exemplo
python3 -m venv .venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

A aplicação sobe em `http://localhost:5000`. Usuários de teste já vêm
cadastrados no banco SQLite (`taskflow.db`, criado automaticamente):

| Usuário | Senha    |
|---------|----------|
| admin   | admin123 |
| aluno   | senha123 |

## Como executar com Docker

```bash
cd app-exemplo
docker build -t taskflow:vuln .
docker run -p 5000:5000 taskflow:vuln
```

## Estrutura

```
app-exemplo/
├── app.py              # Aplicação Flask (versão vulnerável, linha de base)
├── requirements.txt    # Dependências com CVEs conhecidas (uso proposital)
├── Dockerfile           # Dockerfile inseguro (uso no Módulo 2 - Hardening)
└── README.md            # Este arquivo
```

Cada módulo cria, dentro da sua própria pasta `codigo/`, uma cópia ou um
patch desta aplicação demonstrando o "antes" (vulnerável) e o "depois"
(corrigido) referente ao tema daquele encontro.

## Exercício

# Análise de Segurança - TaskFlow

### O que é "Shift-Left Security"?
A expressão "shift-left security" refere-se à prática de antecipar as etapas de segurança no ciclo de vida de desenvolvimento de software (SDLC). Em vez de deixar as análises de vulnerabilidades e os testes de segurança apenas para as fases finais (como testes ou produção, à "direita" da linha do tempo), a segurança é "deslocada para a esquerda", ou seja, para o início do projeto. Isso significa incorporar requisitos e testes de segurança desde as fases de planejamento, design e durante a própria escrita do código. Essa abordagem cria uma cultura onde a segurança é pensada de forma contínua, evitando surpresas desagradáveis no final.

### Vulnerabilidade Observada: SQL Injection
Durante a análise do código do TaskFlow, identifiquei uma vulnerabilidade conhecida como **SQL Injection** (Injeção de SQL). De forma simples, isso acontece quando o sistema recebe um texto digitado pelo usuário (como em um campo de login ou de busca) e o envia diretamente para o banco de dados sem fazer nenhuma verificação ou limpeza. 

Isso é um problema grave porque um usuário mal-intencionado pode digitar um "comando" de banco de dados disfarçado de texto comum. Quando o sistema junta isso e envia para o banco, o banco de dados acaba executando esse comando invasor. O resultado pode ser desastroso: o invasor pode conseguir acessar dados sigilosos de outros usuários, alterar informações importantes ou até mesmo apagar o banco de dados inteiro.

### O risco de deixar a segurança para o final
Na minha opinião, esperar até o fim do desenvolvimento para pensar em segurança é uma aposta muito arriscada. Quando falhas estruturais são descobertas apenas nas vésperas de entregar o projeto, corrigi-las costuma ser muito mais difícil, demorado e caro, pois pode exigir que a equipe reescreva grandes partes do código. Além disso, com a pressão para entregar o sistema no prazo, existe o risco de a equipe decidir ignorar algumas falhas para lançar logo o produto, expondo os usuários a ataques e colocando a reputação do projeto em risco. Integrar a segurança desde o primeiro dia evita que problemas pequenos se tornem falhas catastróficas.
# Teste de Pull Request para validar o CI
