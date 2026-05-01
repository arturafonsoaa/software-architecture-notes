# Convenções de Nomenclatura e Cases no PHP

## Definição do conceito

Convenções de nomenclatura são regras consistentes para nomear classes, métodos, variáveis, arquivos e constantes.

Quando falamos em “cases”, estamos falando do formato visual desses nomes, por exemplo:

* `snake_case`
* `camelCase`
* `PascalCase`
* `UPPER_SNAKE_CASE`
* `kebab-case`

No contexto de PHP, essas convenções reduzem ambiguidade e aumentam legibilidade em todo o projeto.

Mais do que estética, nomenclatura é uma decisão de arquitetura de código: ela impacta manutenção, onboarding e consistência técnica.

---

## Problema que ele resolve

Sem convenções claras, times acabam com código assim no mesmo projeto:

* `get_user_data()`
* `getUserData()`
* `GetUserData()`
* `GET_USER_DATA`

Consequências comuns:

* Leitura mais lenta
* Dificuldade para buscar código
* Inconsistência entre módulos
* Revisões com discussões repetitivas
* Maior custo de onboarding

Quando cada pessoa nomeia de um jeito, o sistema perde previsibilidade.

---

## Exemplos práticos em PHP

### Formatos mais comuns

```php
<?php

$customer_name = 'Ana';           // snake_case
$customerName = 'Ana';            // camelCase

class PaymentService {}           // PascalCase

const MAX_RETRY_ATTEMPTS = 3;     // UPPER_SNAKE_CASE
```

---

### Convenções recomendadas em projetos PHP

```php
<?php

declare(strict_types=1);

namespace App\Services\Billing;

final class InvoiceCalculator // PascalCase para classe
{
    private const DEFAULT_TAX_RATE = 0.1; // UPPER_SNAKE_CASE para constante

    public function calculateTotal(float $subtotalAmount): float // camelCase para método
    {
        $taxAmount = $subtotalAmount * self::DEFAULT_TAX_RATE; // camelCase para variável

        return $subtotalAmount + $taxAmount;
    }
}
```

Resumo prático:

* Classes e interfaces: `PascalCase`
* Métodos e propriedades: `camelCase`
* Variáveis: `camelCase`
* Constantes: `UPPER_SNAKE_CASE`
* Namespaces: segmentos em `PascalCase`

---

### Sobre arquivos e estrutura

Em PHP moderno, a convenção costuma seguir PSR-4:

* Nome do arquivo igual ao nome da classe
* Diretórios espelhando o namespace

Exemplo:

* `App\Services\Billing\InvoiceCalculator`
* Arquivo: `app/Services/Billing/InvoiceCalculator.php`

---

### Onde `snake_case` ainda é comum

Mesmo em projetos orientados a `camelCase` no código PHP, `snake_case` ainda aparece em contextos específicos:

* Nomes de colunas em banco (`created_at`, `updated_at`)
* Campos de payload legado
* Alguns contratos externos

O importante é separar contexto interno e externo com clareza.

---

## Convenções específicas de times e empresas

Além de PSRs, empresas geralmente definem regras próprias para reduzir ruído no dia a dia.

Exemplos reais de decisões internas úteis:

* Sufixo obrigatório para serviços (`UserService`, `InvoiceService`)
* Prefixo para interfaces (`UserRepositoryInterface`)
* Proibição de abreviações ambíguas (`qty`, `cfg`, `usr`)
* Padrão para nomes de eventos de domínio (`UserRegistered`, `InvoicePaid`)
* Padrão para testes (`it_calculates_total_with_default_tax`)

Essas convenções não existem para “engessar”, mas para reduzir decisões repetitivas de baixo valor.

### Como formalizar bem

Uma forma prática de consolidar isso:

* Definir guia curto de nomenclatura no repositório
* Exemplificar casos aceitos e não aceitos
* Automatizar parte das regras com linters e code review
* Revisar convenções quando o contexto do produto mudar

Se a regra não melhora clareza, ela vira burocracia.

---

## Quando usar

* Em qualquer projeto com mais de uma pessoa
* Em codebases com crescimento contínuo
* Em times com onboarding frequente
* Em sistemas com múltiplos módulos
* Quando legibilidade e manutenção são prioridade

---

## Quando não usar (ou usar com cautela)

* Para impor regras sem contexto
* Para bloquear entregas por detalhes irrelevantes
* Para forçar padrões incompatíveis com bibliotecas externas
* Para criar exceções demais e perder consistência

Convenção boa simplifica. Convenção ruim só adiciona atrito.

---

## Trade-offs envolvidos

**Vantagens:**

* Código mais legível
* Menos ambiguidades
* Revisões mais objetivas
* Onboarding mais rápido
* Maior previsibilidade arquitetural

**Desvantagens:**

* Exige disciplina do time
* Pode gerar discussões iniciais de padronização
* Pode virar burocracia se mal aplicada
* Em legados, migração pode ter custo alto

O ganho aparece no médio e longo prazo, principalmente em times maiores.

---

## Resumo final

Escolher entre `snake_case`, `camelCase` e `PascalCase` é só uma parte do problema.

O principal é definir um padrão coerente por contexto e aplicá-lo com consistência.

Em PHP, a base costuma ser:

* `PascalCase` para classes
* `camelCase` para métodos e variáveis
* `UPPER_SNAKE_CASE` para constantes

A partir daí, cada empresa pode ajustar detalhes conforme domínio, framework e cultura técnica.

A regra central continua a mesma: nomenclatura deve reduzir fricção, não criar complexidade.
