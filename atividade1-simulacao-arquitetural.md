# Arquitetura Orientada a Serviços — SOA e Web Services
## Atividade 1 — Simulação Arquitetural

---

## Enunciado

**Sistema Universitário:** Times: Financeiro, Matrícula, Biblioteca, RH.
**Problema:** Todos precisam de autenticação. Você criaria um componente compartilhado ou cada domínio possui sua solução? Justifique sua resposta.

---

## Resposta

### Decisão: Componente Compartilhado de Autenticação

A escolha adequada nesse cenário é a criação de um **serviço centralizado de autenticação**, acessível por todos os domínios do sistema universitário.

---

### Justificativa

#### 1. Princípio da Reusabilidade (core do SOA)

O pilar fundamental da Arquitetura Orientada a Serviços é a reutilização de serviços entre múltiplos consumidores. A autenticação é um **serviço horizontal** — não pertence a nenhum domínio específico, mas é demandada por todos. Implementá-la quatro vezes viola diretamente esse princípio e multiplica o custo sem agregar valor de negócio.

#### 2. Consistência e Segurança

Com soluções isoladas por domínio, cada time adotaria sua própria lógica de validação, armazenamento de credenciais, políticas de senha e controle de sessão. Isso cria superfícies de ataque distintas e inconsistentes. Um serviço compartilhado centraliza a política de segurança, facilitando auditorias, rotação de credenciais e conformidade com regulamentações (ex.: LGPD).

#### 3. Manutenção e Evolução

Se uma vulnerabilidade for descoberta ou uma política precisar mudar (ex.: MFA obrigatório), a correção ocorre em um único ponto. Com quatro implementações independentes, a mesma mudança exigiria quatro intervenções simultâneas, aumentando o risco de inconsistência e janelas de exposição.

#### 4. Separação de Responsabilidades

Cada domínio deve focar em sua lógica de negócio:

| Domínio    | Responsabilidade Principal                  |
|------------|---------------------------------------------|
| Financeiro | Cobranças, boletos, inadimplência           |
| Matrícula  | Gestão de disciplinas e vínculos acadêmicos |
| Biblioteca | Acervo, empréstimos, reservas               |
| RH         | Folha de pagamento, contratos, benefícios   |

Autenticação não é responsabilidade de nenhum deles — é uma **preocupação transversal (cross-cutting concern)**.

#### 5. Single Sign-On (SSO)

Um serviço centralizado permite que o usuário se autentique uma única vez e acesse todos os sistemas sem nova autenticação. Isso melhora a experiência e reduz pontos de falha.

---

### Arquitetura Proposta

```
┌─────────────────────────────────────────────────────────┐
│                  ESB / API Gateway                      │
└────────────┬──────────┬──────────┬─────────────┬────────┘
             │          │          │             │
     ┌───────▼──┐ ┌─────▼────┐ ┌───▼───────┐ ┌───▼────┐
     │Financeiro│ │Matrícula │ | Biblioteca│ │   RH   │
     └───────┬──┘ └─────┬────┘ └──┬────────┘ └────┬───┘
             │          │         │               │
             └──────────┴────┬────┴───────────────┘
                             │  Token JWT / SAML
                    ┌────────▼────────┐
                    │  Serviço de     │
                    │  Autenticação   │
                    │  (centralizado) │
                    └─────────────────┘
```

O fluxo funciona da seguinte forma:
1. O usuário (aluno, professor ou funcionário) realiza login **uma única vez** no serviço de autenticação.
2. O serviço emite um token (ex.: **JWT**) com as identidade e permissões do usuário.
3. Cada domínio valida o token sem precisar reimplementar a lógica de autenticação.
4. O ESB ou API Gateway pode fazer a validação do token antes mesmo de a requisição chegar ao serviço de destino.

---

### Quando a solução por domínio seria aceitável?

A autenticação por domínio só seria justificável se:

- Os domínios pertencessem a **organizações distintas** com requisitos regulatórios incompatíveis.
- Houvesse necessidade de **isolamento total** (ex.: sistema classificado com rede air-gapped).
- Os domínios operassem em **tecnologias radicalmente diferentes** sem capacidade de integração.

Nenhuma dessas condições se aplica a um sistema universitário integrado.

---

### Conclusão

Em um ambiente SOA, serviços compartilhados são a norma, não a exceção. A autenticação é o exemplo mais clássico de serviço candidato à centralização: alta demanda transversal, lógica uniforme e impacto crítico em segurança. Criar quatro implementações independentes seria um anti-padrão que contradiz os fundamentos da arquitetura orientada a serviços.

---

- *Disciplina: Arquitetura Orientada a Serviços — SOA e Web Services*.
- *Professor: Luiz Santos*.
- *Aluno: Belicio Batista Cardoso*.
- *Data: 22 de maio de 2026*.
