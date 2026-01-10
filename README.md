# Standards - LiveMode Tech

Repositório centralizado de padrões de desenvolvimento, boas práticas e guidelines técnicas para projetos da LiveMode.

## 📚 Sobre

Este repositório contém documentação técnica padronizada para garantir consistência, qualidade e boas práticas em todos os projetos da organização.

## 📁 Estrutura

```
standards/
├── README.md                    # Este arquivo
├── development/                 # Padrões de desenvolvimento
│   ├── development-standards-javascript.md
│   └── development-standards-python.md
│   ├── api-guidelines.md
│   └── code-review-checklist.md
├── architecture/                # Padrões arquiteturais
│   ├── project-structure.md
│   └── database-guidelines.md
├── security/                    # Guidelines de segurança
│   ├── secrets-management.md
│   └── authentication-patterns.md
├── testing/                     # Padrões de testes
│   ├── testing-strategy.md
│   └── test-coverage.md
├── ai-dev/                      # Guidelines de AI Dev Tools
│   └── ai-dev-guidelines.md
└── templates/                   # Templates reutilizáveis
    ├── pr-template.md
    ├── commit-message-template.md
    └── api-documentation-template.md
```

## 🚀 Início Rápido

### Para Desenvolvedores

1. **Leia primeiro**:
   - Para projetos JavaScript/TypeScript: [`development-standards-javascript.md`](./development/development-standards-javascript.md)
   - Para projetos Python: [`development-standards-python.md`](./development/development-standards-python.md) (em construção)
2. **Stack obrigatória**: Next.js, React, Radix/shadcn/ui, PostgreSQL, pnpm/bun
3. **Antes de começar**: Verifique os padrões relevantes ao seu projeto

### Para Code Reviewers

- Use o checklist de code review (em breve)
- Verifique conformidade com os padrões estabelecidos
- Consulte guidelines específicas quando necessário

## 📖 Documentação Principal

### Desenvolvimento
- [Padrões de Desenvolvimento JavaScript](./development/development-standards-javascript.md) - Guia completo de stack, APIs REST, HTTP, erros, logging e mais
- [Padrões de Desenvolvimento Python](./development/development-standards-python.md) - Em construção

### AI Dev Tools
- [Guidelines de AI Dev Tools](./ai-dev/ai-dev-guidelines.md) - Recomendações e boas práticas para uso de Cursor, Claude Code, GitHub Copilot e similares

### Arquitetura
- [Estrutura de Projetos](./architecture/project-structure.md) - Princípios básicos de organização e separação de responsabilidades

### Em Breve
- Padrões de Segurança
- Estratégia de Testes
- Templates de Documentação

## 🔄 Como Contribuir

### Adicionar Novo Padrão

1. Crie um novo arquivo na pasta apropriada
2. Siga o formato markdown estabelecido
3. Inclua exemplos práticos quando possível
4. Abra um PR com:
   - Descrição clara do padrão
   - Justificativa para a mudança
   - Exemplos de uso

### Atualizar Padrão Existente

1. Abra um PR explicando a mudança
2. Inclua contexto sobre por que a mudança é necessária
3. Atualize exemplos se necessário
4. Notifique a equipe sobre mudanças significativas

### Processo de Aprovação

- PRs requerem aprovação de pelo menos 2 membros do time
- Mudanças críticas requerem discussão prévia
- Após aprovação, o padrão entra em vigor imediatamente

## 📋 Checklist de Conformidade

### Antes de iniciar um novo projeto:
- [ ] Stack tecnológica está conforme os padrões
- [ ] Linters e formatters configurados
- [ ] Estrutura de pastas definida
- [ ] Ambiente de desenvolvimento configurado

### Durante o desenvolvimento:
- [ ] Tratamento de erros implementado
- [ ] Logging configurado
- [ ] Crash reporting integrado
- [ ] APIs seguem padrões REST
- [ ] Status codes HTTP corretos
- [ ] Documentação de APIs atualizada

## 🔗 Links Úteis

- [Next.js Documentation](https://nextjs.org/docs)
- [shadcn/ui Components](https://ui.shadcn.com)
- [Radix UI Primitives](https://www.radix-ui.com)
- [PostgreSQL Documentation](https://www.postgresql.org/docs)
- [pnpm Documentation](https://pnpm.io)

## 📝 Versionamento

Este repositório segue [Semantic Versioning](https://semver.org/) para mudanças significativas:

- **MAJOR**: Mudanças que quebram compatibilidade
- **MINOR**: Novos padrões ou adições que não quebram compatibilidade
- **PATCH**: Correções e melhorias em padrões existentes

## ❓ FAQ

### Posso desviar dos padrões em casos específicos?

Sim, mas é necessário:
1. Documentar a justificativa
2. Obter aprovação do time
3. Atualizar este repositório se o desvio se tornar padrão

### Como reportar um padrão que não está funcionando?

1. Abra uma issue descrevendo o problema
2. Proponha uma solução alternativa
3. Participe da discussão para encontrar o melhor caminho

### Com que frequência os padrões são atualizados?

- Revisão trimestral obrigatória
- Atualizações podem ocorrer a qualquer momento via PR
- Mudanças críticas são comunicadas via email/slack

## 📞 Contato

Para dúvidas ou sugestões:
- Abra uma issue neste repositório
- Entre em contato com o time de Tech Lead
- Discuta no canal #tech-standards no Slack

## 📅 Histórico de Mudanças

### 2024-01
- Criação do repositório
- Estabelecimento dos padrões iniciais de desenvolvimento
- Definição da stack tecnológica obrigatória

---

**Última atualização**: Janeiro 2024
**Mantido por**: Time de Engenharia LiveMode
