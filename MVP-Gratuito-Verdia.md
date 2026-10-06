# Verdia — MVP com ferramentas gratuitas

**Plano de implementação · Versão 1.1 · 5 de outubro de 2026**

Esta é a proposta vigente para começar o Verdia com baixo custo. Ela substitui, para o MVP, a infraestrutura AWS e o orçamento de equipe do SDD original. O SDD completo continua como referência de funcionalidades, regras clínicas e evolução; sua infraestrutura ampliada não é pré-requisito para este plano.

## 1. Recomendação

Construir um MVP pequeno com **React + TypeScript, Cloudflare Pages/Functions e Supabase Free**, mantendo autenticação forte, permissões, separação entre clínicas e histórico de alterações. Publicar a documentação estática no **GitHub Pages**, em repositório separado do futuro aplicativo.

**Custo de infraestrutura da demonstração: R$ 0/mês, dentro das franquias**, usando subdomínios gratuitos e dados fictícios. Trabalho de desenvolvimento, operação, computador, internet e eventuais serviços externos não se tornam gratuitos por causa disso.

É possível preservar bons controles de confidencialidade e integridade no plano gratuito. A maior concessão é de disponibilidade e recuperação: o Supabase Free pode pausar por inatividade e não inclui backup automático. Portanto, usar primeiro em demonstração e homologação. Um piloto administrativo com dados reais exige os controles e o aceite de risco descritos adiante; para prontuário clínico como fonte principal, a recomendação é migrar para banco com backup gerenciado e manter cópia independente.

**Primeira entrega:** pacientes, agenda e retornos. **Segunda entrega:** anamnese e evolução simples, inicialmente com dados fictícios. Fotos clínicas, IA, estoque, financeiro completo, WhatsApp automático e aplicativo nativo ficam fora do MVP gratuito.

## 2. Escopo enxuto

| Entra no MVP | Definição de pronto |
|---|---|
| Acesso por convite | Usuário autorizado, segundo fator TOTP e vínculo à clínica |
| Cadastro de pacientes | Nome, nascimento quando necessário, contato, responsável e preferências; CPF opcional |
| Agenda | Profissional, início/fim, bloqueio, cancelamento, falta e reagendamento |
| Retornos | Data/prazo definido pelo profissional, responsável e lista de pendências |
| Anamnese básica | Um template revisado, versionado e com campos essenciais |
| Evolução | Rascunho, finalização explícita e adendo; sem edição silenciosa do finalizado |
| Exportação | Pacientes em CSV e prontuário em formato legível, com autoria e datas |
| Auditoria | Alterações e leituras clínicas registradas no servidor |
| Backup/restore | Cópia cifrada fora do projeto e restauração demonstrada |

Começar com **uma clínica, até cinco usuários e até mil pacientes**, como limite operacional proposto, não como garantia de capacidade do fornecedor. Testar isolamento com uma segunda clínica fictícia desde o início, para evitar uma migração insegura quando o produto crescer.

Recepção vê cadastro, agenda e tarefas; profissional vê conteúdo clínico autorizado. Gestor administrativo não recebe acesso clínico automaticamente. Não disponibilizar cadastro público ou portal do paciente nesta primeira versão.

## 3. Ferramentas e custo

| Componente | Escolha | Custo inicial e limitação |
|---|---|---|
| Interface responsiva | React, TypeScript e Vite | Sem licença paga; componentes acessíveis e CSS próprio |
| Hospedagem da interface | Cloudflare Pages Free | R$ 0 dentro das franquias; subdomínio pages.dev |
| API/BFF pequeno | Pages Functions, runtime Workers | Franquia gratuita; funções devem ser curtas e medidas |
| Banco | Supabase Free / PostgreSQL | 500 MB de banco por projeto; monitorar crescimento |
| Autenticação | Supabase Auth + TOTP | Segundo fator por aplicativo, sem dependência de SMS |
| Arquivos | Adiados no MVP | Evita custo, varredura e complexidade de fotos/prontuários anexados |
| Código do aplicativo | GitHub, repositório privado | Plano gratuito; segredos fora do código |
| Documento compartilhável | GitHub Pages, repositório público de documentação | Publicação estática gratuita nas condições do GitHub Free |
| Testes | Vitest, Playwright, axe-core e pgTAP | Execução local gratuita; CI sujeito à franquia |
| Design e planejamento | Penpot ou Figma Free; GitHub Issues | Usar recursos gratuitos; verificar limites do plano escolhido |
| Backup | pg_dump/Supabase CLI + age ou ferramenta equivalente | Ferramentas gratuitas; exige destino cifrado, agenda e responsável |
| E-mail de conta | SMTP externo gratuito, quando configurado | Domínio verificado e franquia do provedor podem limitar custo zero |

O plano Supabase Free consultado informa também 1 GB de arquivos, duas instâncias ativas e pausa após uma semana sem atividade. Pro parte de US$ 25/mês e inclui backups diários de banco com sete dias de retenção; recuperação pontual tem cobrança separada. Esses recursos não substituem governança nem backup dos objetos. [Supabase — preços](https://supabase.com/pricing).

Cloudflare Pages Free permite 500 builds por mês. Functions consome a franquia de Workers: 100 mil requisições/dia e limite de CPU de 10 ms por requisição no plano gratuito. Isso exige medir autenticação, validação e exportações: exceder CPU pode interromper a função mesmo com poucas visitas. [Pages — limites](https://developers.cloudflare.com/pages/platform/limits/), [Workers — limites](https://developers.cloudflare.com/workers/platform/limits/).

## 4. Arquitetura do MVP

```text
Celular / tablet / PC
         │ HTTPS
         ▼
Cloudflare Pages: React estático
         │ /api, mesma origem
         ▼
Pages Functions: autenticação, autorização e validação
         │ token do usuário; operações curtas
         ▼
Supabase: Auth + PostgreSQL + RLS + funções SQL transacionais
         │
         └── exportação de backup → cópia cifrada independente

GitHub Pages → apenas documentação pública, sem login de pacientes
```

O backend é pequeno e organizado por módulos: acesso, pacientes, agenda, retornos e prontuário. O PostgreSQL concentra as transações de agenda, finalização e adendos. Não implantar ECS, Kubernetes, Redis, fila dedicada, servidor NestJS ou banco vetorial nesta etapa.

Escolher São Paulo para o projeto Supabase quando disponível no provisionamento. Cloudflare e serviços auxiliares podem processar dados fora do Brasil; mapear contratos e transferências antes do uso real. Não afirmar residência exclusivamente brasileira de ponta a ponta. [Supabase — regiões](https://supabase.com/docs/guides/platform/regions).

**Sessão:** BFF mantém tokens fora do JavaScript do navegador, em cookies HttpOnly, Secure e SameSite, protegidos contra adulteração; usar proteção CSRF e checagem de origem. Não persistir prontuário ou tokens em localStorage. Renovação ocorre no servidor. Não assumir que o SDK padrão já implementa exatamente essa estratégia: testar login, refresh concorrente, expiração, logout e recuperação como parte da primeira fase.

**Permissões:** chamadas normais ao Supabase usam o token do usuário e RLS. Não usar service_role como atalho geral: ela ignora RLS. Chaves administrativas ficam apenas em funções administrativas restritas, com segredos de ambiente e escopo mínimo. Testar o acesso direto à API do Supabase, pois esconder uma rota no frontend não protege o banco. [Supabase — RLS](https://supabase.com/docs/guides/database/postgres/row-level-security).

**Modelo mínimo:** Organization, Membership, Patient, Professional, Appointment, ReturnTask, FormTemplateVersion, Encounter, ClinicalEntry, Addendum, AuditEvent e SessionRevocation. Todas as tabelas clínicas têm tenant_id e referências que não aceitam relações entre clínicas.

Para leitura clínica auditada, não permitir SELECT direto nas tabelas-base pelos papéis cliente. Expor uma operação SQL restrita que valida usuário, vínculo ativo, MFA e acesso ao paciente, registra a leitura e retorna apenas os campos autorizados. Se SECURITY DEFINER for necessário, usar proprietário sem BYPASSRLS, FORCE RLS quando aplicável, search_path fixo, autorização explícita e testes negativos; revogar execução pública. RLS sozinha não produz auditoria de SELECT. Views e funções devem ser revisadas, não apenas tabelas.

## 5. Segurança que permanece obrigatória

| Controle | Implementação mínima | Teste de aceite |
|---|---|---|
| Segundo fator | TOTP obrigatório para equipe; exigir nível aal2 na autorização | Senha correta sem MFA não abre dados assistenciais |
| Isolamento | tenant_id, RLS e vínculo ativo no banco | Usuário da clínica A não lê/altera nenhum recurso da B |
| Recepção segregada | Papéis e rotas/consultas com campos mínimos | Recepção recebe negação para anamnese e evolução |
| Sessões revogáveis | Verificação de vínculo e revogação da sessão no servidor/banco | Remover usuário ou revogar sessão bloqueia uso, inclusive com token ainda válido |
| Agenda consistente | Constraint de intervalo e transação | Duas reservas simultâneas não ocupam o mesmo recurso |
| Histórico íntegro | Finalização imutável e adendo | UPDATE direto no registro finalizado falha |
| Auditoria confiável | Eventos criados pelo servidor; sem UPDATE/DELETE pelo usuário | Ator não consegue falsificar autoria nem remover eventos |
| Conteúdo seguro | CSP, sem HTML arbitrário, dependências revisadas e limites de entrada | Texto malicioso não executa script |
| Segredos | Apenas variáveis protegidas; nenhum segredo no build/README | Varredura do repositório e artefato publicado |
| Privacidade | Sem replay de sessão, analytics clínico ou payloads sensíveis em logs | Inspeção de logs e erros |
| Backups | Cifrados, independentes e com recuperação ensaiada | Recuperar conjunto de teste e reconciliar registros |

A configuração de MFA exige interface de inscrição, desafio e recuperação; não basta habilitar a opção no painel. O backend e as políticas devem exigir o nível correto. [Supabase — MFA](https://supabase.com/docs/guides/auth/auth-mfa).

Não compartilhar logins, não tornar buckets públicos e não desativar RLS para resolver erros. Bloquear acesso anônimo e autoinscrição. Não confiar em tenant_id ou papéis editáveis enviados pelo cliente. Aplicar cabeçalhos no-cache às respostas privadas, CORS restrito e limites de tentativas. Proteções contra abuso também devem ser verificadas nos endpoints públicos de autenticação do fornecedor.

MFA no GitHub, Cloudflare e Supabase para os administradores. Um repositório privado ajuda a restringir exposição do projeto, mas não substitui a proteção de credenciais e dados.

## 6. O que o gratuito não entrega

### Disponibilidade

Não vender SLA ou atendimento 24×7 nesta fase. A pausa por inatividade é uma condição do plano, não um problema a contornar com acessos artificiais. Registrar um procedimento de retomada e um fluxo manual de contingência. [Supabase — pausa de projetos](https://supabase.com/docs/guides/platform/free-project-pausing).

### Recuperação

O free não inclui backup automático gerenciado. Criar exportação lógica diária e antes de migrações; manter sete versões diárias e quatro semanais como política inicial, sujeita à governança de retenção. Criptografar antes de enviar para destino privado independente e manter chave de recuperação separada. O GitHub, mesmo privado, não é destino para dumps de pacientes.

O backup deve abranger esquemas, dados, políticas e funções, além de procedimento documentado para identidades/Auth, associações, configurações e segredos. Um dump parcial de tabelas do app não equivale à restauração integral. Executar restore em projeto de teste, conferindo usuários, vínculos e histórico; não duplicar tokens/sessões de produção.

Meta proposta para piloto: RPO de até 24 horas e RTO de até um dia útil, apenas depois de demonstrados. Uma tarefa em computador desligado falha: se não houver executor confiável e responsável verificando resultado diariamente, o gate de dados reais não passa. O custo de armazenamento/executor deve ser incluído se não houver recurso existente adequado. A documentação oficial recomenda exportações externas para o plano gratuito. [Supabase — backups](https://supabase.com/docs/guides/platform/backups).

### Mensagens e login

O SMTP padrão do Supabase é destinado a testes, tem restrição de destinatários e não deve ser base para convites e recuperação em produção. Configurar SMTP externo e testar entrega, limites, domínio e recuperação. Não desabilitar confirmação/recuperação para economizar. [Supabase — SMTP](https://supabase.com/docs/guides/auth/auth-smtp).

Se não houver domínio/provedor adequado, manter demonstração com contas de teste permitidas; a fase com usuários reais depende dessa configuração. Um domínio próprio pode ser o primeiro custo anual. Lembretes do MVP são tarefas dentro do sistema e contato manual da clínica, sem bot não oficial de WhatsApp.

## 7. Fases e pequenas entregas

| Fase | Prazo estimado | Entrega | Aceite |
|---|---|---|---|
| M0 — Fundação | Semana 1 | Projetos, dados fictícios, MFA, papéis, tenant e backup inicial | Testes de isolamento e restore mínimo passam |
| M1 — Pacientes | Semana 2 | Cadastro, busca, responsáveis e preferências | Recepção usa sem acessar campos clínicos |
| M2 — Agenda | Semanas 3–4 | Reserva, bloqueio, cancelamento e falta | Concorrência não duplica horário; histórico preservado |
| M3 — Retornos | Semana 5 | Lista por prazo, responsável e status | Reagendamento/cancelamento mantém tarefas coerentes |
| M4 — Prontuário básico | Semanas 6–7 | Template, atendimento, rascunho, finalização e adendo | Finalizado não muda; profissional revisa o modelo |
| M5 — Liberação | Semana 8 | Exportação, restore completo, revisão de segurança e treinamento | Checklist de liberação aprovado; plano de contingência testado |

**Calendário de referência: 6–8 semanas com um desenvolvedor experiente dedicado**, apoio clínico semanal e escopo estável. Estimativa própria: 240–360 horas técnicas totais; dedicação parcial ou aprendizado da stack aumenta o calendário. Não é compromisso de entrega. O cronograma depende especialmente do BFF/autenticação e da revisão clínica.

Piloto administrativo pode começar após M3 e aprovação dos controles correspondentes. M4 começa com dados fictícios. A agenda e o cadastro já envolvem dados pessoais reais; “administrativo” não significa ausência de obrigações de privacidade.

Para cada fase, demonstrar uma jornada funcionando em celular, tablet e PC. Limitar opções: um template inicial, uma unidade, cinco usuários e sem importação de acervo clínico legado.

## 8. Custos e quando pagar

| Cenário | Infraestrutura prevista | Condição |
|---|---|---|
| Documento online | R$ 0/mês no GitHub Pages | Repositório público apenas com documentação |
| Demonstração do MVP | R$ 0/mês em cloud | Franquias respeitadas, dados fictícios, subdomínios gratuitos |
| Piloto administrativo gratuito | Pode permanecer em R$ 0/mês de planos cloud | Backup independente, SMTP/domínio já disponíveis e risco operacional aceito |
| Pequena produção recomendada | Supabase Pro a partir de US$ 25/mês + extras necessários | Backup gerenciado, sem pausa do Free; ainda sem prometer SLA empresarial |
| CPU/API excede plano gratuito | Cotar plano pago do runtime | Não remover verificação/autorização para caber no limite |

US$ 25 equivalem a R$ 137,50 no cenário hipotético de R$ 5,50 por dólar, antes de tributos/extras. Não é cotação cambial. O plano base não inclui automaticamente PITR, armazenamento externo, domínio, mensageria ou suporte operacional.

Para desenvolvimento próprio, o desembolso com licença pode ser zero, mas há 240–360 horas de trabalho estimado. Se terceirizado à mesma hipótese de R$ 140–220/h do SDD, equivale a **R$ 33.600–79.200**, antes de reserva, revisão independente e operação. Ferramentas gratuitas reduzem a conta cloud; reduzir escopo é o que mais reduz o custo de desenvolvimento.

Gatilhos propostos: revisar plano aos 70% da franquia e migrar antes de chegar a 85%; falha de backup exige correção imediata; necessidade de disponibilidade diária ou prontuário como fonte única exige rever o plano; qualquer falha repetida de CPU requer ajuste/upgrade. Monitorar tamanho do banco, saída de dados, quantidade de chamadas, CPU, erros e resultado das cópias. Definir responsável semanal.

## 9. Critérios para dados reais

Antes de cadastrar pacientes reais, demonstrar: MFA, recuperação de acesso, isolamento entre clínicas, segregação da recepção, proteção do repositório, backup/restauração, autorização de exportação e contingência. Mapear finalidade, hipótese legal, fornecedores, retenção e direitos dos titulares; aprovar os fluxos com responsáveis da clínica.

Antes do prontuário clínico real: validar modelo de anamnese, autoria/finalização, assinatura adequada ao fluxo, guarda, exportação completa, leitura auditada e retificação. Recomenda-se backup gerenciado mais cópia independente. Não declarar conformidade automática com LGPD, conselho profissional ou requisitos documentais por usar um fornecedor conhecido. As obrigações e referências legais estão no SDD completo.

Revisão humana e controles locais continuam necessários em qualquer plano. Não há obrigação técnica de pagar por toda biblioteca; há requisitos de operação que uma franquia gratuita pode não atender.

## 10. O que foi adiado para economizar

Fotos e anexos ficam fora até haver armazenamento, varredura e backup adequados. IA, OCR, voz e interpretação de imagens ficam fora por custo e validação. Financeiro começa posteriormente por contas simples; estoque e rastreio de lotes depois. WhatsApp será via API oficial em fase própria. Nenhuma automação deve substituir revisão clínica.

O caminho de evolução preferido é ampliar o plano gerenciado e otimizar consultas antes de reconstruir a infraestrutura. Manter migrações SQL, dados exportáveis e adaptadores facilita mudar de fornecedor futuramente. AWS do SDD permanece uma opção de evolução, não um destino obrigatório.

## 11. Publicação do documento no GitHub

O repositório de documentação terá uma página inicial com este plano, o SDD visual completo, fontes Markdown e os três estudos de logomarca. Publicação estática, sem analytics, sem formulário, sem banco, sem dados de pacientes e sem credenciais.

O repositório público de documentação é distinto do futuro repositório privado do aplicativo. Não publicar a pasta inteira de trabalho, dumps, .env, tokens, screenshots privados ou anexos do chat. Remover identificadores internos de conversas da versão compartilhável.

GitHub Pages hospeda HTML/CSS/JavaScript estáticos. No GitHub Free, a documentação de referência informa disponibilidade para repositórios públicos; repositórios privados dependem do plano. Este uso é para documentação, não para operar o SaaS ou guardar prontuários. [GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages).

O endereço final só pode ser informado como publicado depois de criar/usar o repositório correto, enviar os arquivos, habilitar Pages e verificar o HTML servido. Não confundir arquivo salvo localmente com publicação online.

## 12. Decisão de partida

Adotar **Cloudflare + Supabase** para o MVP, com um backend pequeno, segurança desde M0 e escopo administrativo primeiro. Validar gratuitamente com dados fictícios. Liberar piloto limitado somente após testes e responsabilidade operacional definida. Pagar pelo primeiro recurso que faltar para preservar dados e continuidade, em vez de cortar um controle essencial.

Fontes de planos e limites consultadas em 05/10/2026. Os limites podem mudar. Este documento é um plano, não a implementação do CRM nem uma auditoria de segurança já realizada.
