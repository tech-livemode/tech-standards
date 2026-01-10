# Guidelines de Uso de AI Dev Tools - LiveMode (Em Construção)

> **Status**: Este documento está em construção e será atualizado conforme aprendemos e evoluímos no uso de ferramentas de AI Dev.

Este documento contém recomendações e boas práticas para o uso de ferramentas de AI Dev como Cursor, Claude Code, GitHub Copilot e similares.

## 🎯 Objetivo

As ferramentas de AI Dev são poderosas aliadas no desenvolvimento, mas devem ser usadas de forma consciente e responsável para garantir qualidade, segurança e manutenibilidade do código.

## 🛠️ Ferramentas Recomendadas

Incentivamos o teste constante e comparação entre diferentes soluções de AI Dev para encontrar a que melhor se adequa ao seu fluxo de trabalho.

### Principais Ferramentas
- **Cursor**: Integração completa com editor, contexto do projeto, suporte a múltiplos modelos
- **Claude Code**: Excelente compreensão de contexto, geração de código de alta qualidade
- **GitHub Copilot**: Autocomplete inteligente, sugestões em tempo real, integração nativa com VS Code

### Recomendação
Teste e compare diferentes ferramentas. Cada desenvolvedor pode ter preferências diferentes baseadas em seu estilo de trabalho. O importante é encontrar a ferramenta que aumenta sua produtividade mantendo a qualidade do código.

## ✅ Boas Práticas

### 1. Revisão de Código Gerado
- **SEMPRE revise** o código gerado pela AI antes de commitar
- Entenda o que o código faz, não apenas aceite
- Verifique se segue os padrões do projeto
- Teste o código gerado antes de usar em produção

### 2. Contexto e Prompting
- **Forneça contexto adequado**: Abra arquivos relevantes antes de pedir ajuda
- **Seja específico**: Quanto mais detalhado o prompt, melhor o resultado
- **Use exemplos**: Mostre exemplos do padrão que você quer seguir
- **Referencie padrões**: Mencione os padrões do projeto quando relevante

### 3. Segurança e Privacidade
- **Nunca compartilhe secrets**: Não inclua tokens, senhas ou dados sensíveis em prompts
- **Cuidado com dados privados**: Evite compartilhar dados de usuários ou informações confidenciais
- **Revise imports e dependências**: Verifique se novas dependências são seguras

### 4. Manutenibilidade
- **Código legível**: Prefira código claro e bem documentado
- **Siga padrões do projeto**: Use os padrões estabelecidos (convenções de nome, estrutura, etc)
- **Comente quando necessário**: Adicione comentários explicativos para lógica complexa

### 5. Testes
- **Gere testes quando apropriado**: Use AI para criar testes, mas revise-os
- **Valide testes gerados**: Certifique-se de que os testes realmente testam o comportamento esperado
- **Mantenha cobertura**: Use AI para ajudar a manter boa cobertura de testes

## ⚠️ Cuidados e Limitações

### O que a AI NÃO deve fazer sozinha
- Decisões arquiteturais importantes sem revisão
- Mudanças em lógica crítica de negócio sem validação
- Commits diretos sem revisão humana
- Remoção de código sem entender o impacto

### Limitações conhecidas
- Pode sugerir código desatualizado ou com vulnerabilidades
- Pode não seguir padrões específicos do projeto
- Pode gerar código que funciona mas não é otimizado
- Pode não entender completamente o contexto do negócio

## 📝 Casos de Uso Recomendados

### ✅ Bom uso de AI Dev
- Gerar boilerplate code
- Criar testes unitários
- Refatorar código existente
- Explicar código complexo
- Sugerir melhorias de performance
- Gerar documentação
- Traduzir código entre linguagens
- Encontrar e corrigir bugs simples
- Gerar exemplos de uso de APIs

### ❌ Evite usar AI Dev para
- Decisões de arquitetura sem revisão
- Lógica crítica de negócio sem validação
- Código de segurança sem revisão detalhada
- Substituir completamente o entendimento do código
- Commits automáticos sem revisão

## 🔍 Checklist de Revisão

Antes de usar código gerado por AI:

- [ ] Entendi o que o código faz
- [ ] O código segue os padrões do projeto
- [ ] Testei o código localmente
- [ ] Não há secrets ou dados sensíveis expostos
- [ ] As dependências são seguras e necessárias
- [ ] O código está bem documentado
- [ ] Os testes passam (se aplicável)
- [ ] A performance é aceitável
- [ ] O código é acessível e legível

## 💡 Dicas de Prompting Eficiente

### Estrutura de um bom prompt
1. **Contexto**: O que você está tentando fazer?
2. **Restrições**: Quais são as limitações ou requisitos?
3. **Exemplos**: Mostre exemplos do padrão desejado
4. **Formato**: Como você quer a resposta?

### Exemplos de bons prompts

```
"Refatore esta função para seguir o padrão de tratamento de erros
do nosso projeto. Veja o arquivo lib/api-error.ts como referência."
```

```
"Crie testes unitários para esta função usando Vitest.
Siga o padrão dos testes em __tests__/utils/."
```

```
"Explique esta função complexa e sugira melhorias de performance,
mantendo a compatibilidade com a API existente."
```

### Exemplos de prompts ruins (evite)

```
❌ "Crie uma API"
```
**Problema**: Muito vago, sem contexto, sem requisitos específicos.

```
❌ "Corrige isso"
```
**Problema**: Não especifica o que precisa ser corrigido, sem contexto.

```
❌ "Faz funcionar"
```
**Problema**: Não explica o problema, não fornece contexto do erro.

```
❌ "Melhora o código"
```
**Problema**: Muito genérico, não especifica o que precisa ser melhorado.

```
❌ "Cria um componente React igual ao do projeto X"
```
**Problema**: Não fornece acesso ao projeto X, sem contexto específico.

**Dica**: Sempre forneça contexto, seja específico, e mostre exemplos do padrão que você quer seguir.

## 🚀 Integração com Workflow

### No desenvolvimento diário
1. Use AI para gerar código inicial
2. Revise e ajuste conforme necessário
3. Teste localmente
4. Faça code review (mesmo que seja seu próprio código)
5. Commit seguindo os padrões do projeto

### Em code reviews
- Use AI para ajudar a entender código complexo
- Peça sugestões de melhorias
- Valide se o código gerado segue padrões
- Não aceite código apenas porque foi gerado por AI

## 📚 Recursos Adicionais

- [Cursor Documentation](https://cursor.sh/docs)
- [Claude Code Best Practices](https://www.anthropic.com)
- [GitHub Copilot Documentation](https://docs.github.com/copilot)

## 🔄 Atualizações

Este documento deve ser atualizado conforme:
- Novas ferramentas são adotadas
- Novos padrões são estabelecidos
- Lições aprendidas são identificadas
- Melhores práticas evoluem

---

**Lembre-se**: AI Dev tools são ferramentas poderosas, mas nós desenvolvedores é que somos responsáveis pelo resultado. Usemo-as para aumentar nossa produtividade, não para substituir nosso julgamento e conhecimento.
