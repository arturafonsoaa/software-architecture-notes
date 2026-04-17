# Lei de Demeter (Law of Demeter)

## Definição do conceito

A **Lei de Demeter**, também conhecida como **Princípio do Menor Conhecimento**, afirma que:

> Um método deve interagir apenas com seus colaboradores diretos.

Em termos práticos, um objeto deve chamar métodos apenas de:

* Si mesmo
* Seus próprios atributos
* Objetos recebidos como parâmetro
* Objetos que ele mesmo instancia

Ele não deve navegar profundamente pela estrutura interna de outros objetos.

O objetivo principal é **reduzir acoplamento estrutural** e proteger o encapsulamento.

---

## Problema que ele resolve

Sem a Lei de Demeter, é comum vermos código que depende fortemente da estrutura interna de outros objetos.

Exemplo típico:

```php
$zipCode = $order->getCustomer()->getAddress()->getZipCode();
```

Problemas desse tipo de abordagem:

* Alto acoplamento entre camadas
* Fragilidade a mudanças internas
* Violação de encapsulamento
* Código difícil de manter
* Propagação de conhecimento estrutural

Se a classe `Customer` mudar a forma como gerencia `Address`, todos os pontos que navegam nessa estrutura podem quebrar.

---

## Exemplos práticos em PHP

### ❌ Violação da Lei de Demeter

```php
$zipCode = $order->getCustomer()->getAddress()->getZipCode();
```

O código externo conhece toda a cadeia de relacionamentos.

---

### ✅ Aplicando a Lei de Demeter

Encapsulando o comportamento:

```php
class Order
{
    public function __construct(
        private Customer $customer,
    ) {}

    public function getCustomerZipCode(): string
    {
        return $this->customer->getZipCode();
    }
}

class Customer
{
    public function __construct(
        private Address $address,
    ) {}

    public function getZipCode(): string
    {
        return $this->address->getZipCode();
    }
}
```

Uso:

```php
$city = $order->getCustomerZipCode();
```

Agora apenas `Order` e `Customer` conhecem a estrutura interna, e o código externo não precisa saber sobre `Address`.
---

### Outro exemplo em domínio puro

❌ Antes:

```php
if ($order->getCustomer()->isPremium()) {
    $discount = 0.2;
}
```

✅ Depois:

```php
class Order
{
    public function discountRate(): float
    {
        return $this->customer->isPremium() ? 0.2 : 0.0;
    }
}
```

Uso:

```php
$discount = $order->discountRate();
```

A responsabilidade foi movida para onde faz mais sentido.

---

## Quando usar

* Em regras de negócio
* Em lógica de domínio
* Quando há navegação profunda entre objetos
* Quando você quer reduzir acoplamento estrutural
* Ao modelar entidades com responsabilidades claras

Especialmente útil em sistemas orientados a objetos com múltiplas camadas.

---

## Quando não usar

* Em DTOs simples
* Em estruturas de dados puras
* Em código temporário ou scripts simples
* Quando a abstração criada torna o código artificial ou verboso demais

Aplicar a Lei de Demeter cegamente pode gerar métodos intermediários desnecessários.

---

## Trade-offs envolvidos

**Vantagens:**

* Menor acoplamento
* Melhor encapsulamento
* Código mais estável a mudanças
* Intenção mais clara
* Melhor modelagem de domínio

**Desvantagens:**

* Pode aumentar o número de métodos na classe
* Pode gerar “pass-through methods” (métodos que apenas delegam)
* Pode parecer verboso em modelos muito simples

Como todo princípio arquitetural, trata-se de equilíbrio.

---

## Resumo final

A Lei de Demeter incentiva a redução do conhecimento estrutural entre objetos.

Se um trecho de código depende de muitos `->` encadeados, provavelmente há acoplamento excessivo.

Ao mover responsabilidades para mais perto dos dados e encapsular a navegação interna, o sistema se torna:

* Mais estável
* Mais coeso
* Mais fácil de evoluir

O princípio não é eliminar acesso a relacionamentos, mas evitar que detalhes internos se tornem dependências externas.
