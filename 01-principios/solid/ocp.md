# Open/Closed Principle (OCP)
## Definição do conceito

O **Open/Closed Principle (OCP)** afirma que um módulo de software deve ser:

> aberto para extensão, mas fechado para modificação

Na prática, isso significa projetar o sistema de forma que **novos comportamentos possam ser adicionados sem alterar o código já existente e validado.**

“Fechado para modificação” não significa imutável. Significa que, para determinados tipos de mudança (principalmente variações previsíveis), o código já existente não deve precisar ser alterado repetidamente.

“Aberto para extensão” significa que o sistema deve permitir a adição de novos comportamentos através de:

- Novas classes
- Novas implementações
- Composição de objetos
- Polimorfismo

---

## Problema que ele resolve

Sem OCP, sistemas tendem a evoluir assim:

- Novas funcionalidades são adicionadas modificando classes existentes
- O mesmo arquivo é alterado repetidamente
- Surgem estruturas com `if/else` ou `switch` crescendo sem controle
- Mudanças simples passam a ter alto risco de regressão

### Consequências comuns

1. **Alto acoplamento** \
A lógica central conhece detalhes demais (UI, banco, formatos, integrações).
2. **Baixa manutenibilidade** \
Pequenas mudanças exigem alterar código crítico.
3. **Risco de regressão** \
Alterar algo para adicionar uma feature pode quebrar outra.
4. **Dificuldade de extensão** \
O sistema cresce por “edição” e não por “composição”.

O OCP resolve isso ao **isolar pontos de variação**, permitindo que novas funcionalidades sejam adicionadas sem impactar o núcleo do sistema.

## Exemplos práticos em PHP

### Exemplo ruim (violando OCP)

Cenário: gerar relatório de vendas em diferentes formatos.

```php
<?php

class SalesRepository
{
    public function getAll(): array
    {
        return [
            ['product' => 'Notebook', 'amount' => 3500.00],
            ['product' => 'Mouse', 'amount' => 150.00],
            ['product' => 'Teclado', 'amount' => 250.00],
        ];
    }
}

class SalesReportService
{
    public function __construct(
        private SalesRepository $repository
    ) {}

    public function generate(string $format): string
    {
        $sales = $this->repository->getAll();

        $total = 0;
        foreach ($sales as $sale) {
            $total += $sale['amount'];
        }

        $reportData = [
            'sales' => $sales,
            'total' => $total,
        ];

        if ($format === 'html') {
            $html = "<h1>Relatório</h1>";
            foreach ($reportData['sales'] as $sale) {
                $html .= "<p>{$sale['product']} - {$sale['amount']}</p>";
            }
            $html .= "<strong>Total: {$reportData['total']}</strong>";
            return $html;
        }

        if ($format === 'json') {
            return json_encode($reportData, JSON_PRETTY_PRINT);
        }

        throw new InvalidArgumentException("Formato inválido");
    }
}
```

### Problemas

- A classe `SalesReportService`:
    - Calcula regras de negócio
    - Decide formato
    - Gera HTML
    - Gera JSON
- Cada novo formato exige:
    - Abrir essa classe
    - Adicionar mais if
    - Aumentar risco de erro
- O comportamento cresce via modificação, não extensão.

### Exemplo bom (aplicando OCP)

Separando responsabilidades e isolando o ponto de variação (formato):

```php
<?php

class SalesRepository
{
    public function getAll(): array
    {
        return [
            ['product' => 'Notebook', 'amount' => 3500.00],
            ['product' => 'Mouse', 'amount' => 150.00],
            ['product' => 'Teclado', 'amount' => 250.00],
        ];
    }
}

class SalesReportData
{
    public function __construct(
        public array $sales,
        public float $total
    ) {}
}

interface ReportFormatter
{
    public function format(SalesReportData $reportData): string;
}
```

### Implementações (extensão)

```php
class HtmlReportFormatter implements ReportFormatter
{
    public function format(SalesReportData $data): string
    {
        $html = "<h1>Relatório</h1>";

        foreach ($data->sales as $sale) {
            $html .= "<p>{$sale['product']} - {$sale['amount']}</p>";
        }

        $html .= "<strong>Total: {$data->total}</strong>";

        return $html;
    }
}

class JsonReportFormatter implements ReportFormatter
{
    public function format(SalesReportData $data): string
    {
        return json_encode([
            'sales' => $data->sales,
            'total' => $data->total,
        ], JSON_PRETTY_PRINT);
    }
}
```

### Caso de uso (núcleo estável)

```php
class SalesReportService
{
    public function __construct(
        private SalesRepository $repository,
        private ReportFormatter $formatter
    ) {}

    public function generate(): string
    {
        $sales = $this->repository->getAll();
        $total = array_sum(array_column($sales, 'amount'));

        $data = new SalesReportData($sales, $total);

        return $this->formatter->format($data);
    }
}
```

### Uso

```php
$repository = new SalesRepository();
$formatter = new HtmlReportFormatter(); // pode trocar aqui

$service = new SalesReportService($repository, $formatter);

echo $service->generate();
```

---

### O que mudou

- O comportamento variável (formato) foi isolado
- O núcleo (SalesReportService) não precisa mais ser modificado
- Novos formatos são adicionados criando novas classes

Exemplo de extensão:

```php
class CsvReportFormatter implements ReportFormatter
{
    public function format(SalesReportData $data): string
    {
        $csv = "product,amount\n";

        foreach ($data->sales as $sale) {
            $csv .= "{$sale['product']},{$sale['amount']}\n";
        }

        $csv .= "TOTAL,{$data->total}\n";

        return $csv;
    }
}
```

Nenhuma modificação no serviço principal.

---

## Quando usar

O OCP deve ser aplicado quando existe ***variação previsível***.

### Situações ideais

- Múltiplos formatos (PDF, JSON, CSV, HTML)
- Múltiplos gateways (Stripe, PayPal, etc.)
- Múltiplos canais (email, SMS, push)
- Estratégias de cálculo variáveis
- Integrações externas

### Sinais claros de necessidade

- Crescimento contínuo de if/switch por tipo
- Mesma classe sendo modificada repetidamente
- Funcionalidades crescendo dentro de um único arquivo
- Necessidade frequente de adicionar “mais um caso”

---

## Quando não usar

Aplicar OCP indiscriminadamente gera complexidade desnecessária.

### Evite quando:

- O código é simples e não tende a variar
- Existe apenas um comportamento possível
- Não há histórico ou expectativa de extensão
- A abstração não traz ganho real

### Exemplo de overengineering

```php
interface LoggerInterface
{
    public function log(string $message): void;
}

class FileLogger implements LoggerInterface { ... }
```

Se o sistema nunca terá outro logger, isso é custo sem benefício.

---

## Trade-offs envolvidos

### 1. Aumento de complexidade estrutural
    - Mais classes
    - Mais interfaces
    - Mais indireção
### 2. Custo cognitivo maior
    Entender o fluxo exige navegar por mais arquivos
### 3. Overengineering
    Abstrações prematuras podem atrapalhar
### 4. Mais código para manter
    Cada extensão adiciona novos componentes

---

## Benefícios

### 1. Redução de risco
    Mudanças não afetam código já testado.
### 2. Melhor organização
    Separação clara de responsabilidades.
### 3. Alta extensibilidade
    Novas funcionalidades entram por adição.
### 4. Testabilidade
    Componentes menores e isolados.

---

## Resumo final

O Open/Closed Principle não é sobre evitar mudanças, mas sobre controlar onde as mudanças acontecem.

A ideia central é:

- Identificar o que tende a variar
- Proteger o núcleo do sistema dessas variações
- Permitir crescimento por adição, não por edição

Em termos práticos:

- Se para adicionar uma feature você precisa modificar sempre a mesma classe → provável violação do OCP
- Se você consegue adicionar comportamento criando novas classes que respeitam contratos existentes → OCP bem aplicado

O OCP é uma ferramenta de design para tornar sistemas mais estáveis, previsíveis e evolutivos, desde que aplicado com critério e sem exagero.