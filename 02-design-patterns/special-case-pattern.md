# Padrão de Caso Especial (Special Case Pattern)

## Definição do conceito

O **Padrão de Caso Especial (Special Case Pattern)** consiste em substituir verificações condicionais repetidas por um objeto específico que representa explicitamente um cenário especial.

Em vez de retornar `null` ou espalhar `if` pelo sistema, retorna-se um objeto que:

* Implementa o mesmo contrato da versão “normal”
* Representa um estado especial
* Possui comportamento próprio para esse cenário

Esse padrão é frequentemente associado ao **Null Object Pattern**, sendo considerado uma variação mais intencional e semântica.

---

## Problema que ele resolve

Sistemas orientados a objetos frequentemente acumulam verificações como:

```php
$user = $repository->find($id);

if ($user === null) {
    return 'User not found';
}

return $user->name();
```

Quando esse padrão se repete:

* A lógica condicional se espalha
* A complexidade ciclomática aumenta
* O código fica mais difícil de manter
* O comportamento especial não é modelado explicitamente

O sistema passa a depender de verificações estruturais (`null`) em vez de comportamento polimórfico.

---

## Exemplos práticos em PHP

### ❌ Abordagem com `null`

```php
$subscription = $user->subscription;

if (!$subscription) {
    return 'No active subscription';
}

return $subscription->planName();
```

O chamador precisa decidir o que fazer.

---

### ✅ Aplicando o Padrão de Caso Especial

#### 1. Definindo um contrato

```php
interface Subscription
{
    public function planName(): string;
}
```

---

#### 2. Implementação normal

```php
class ActiveSubscription implements Subscription
{
    public function __construct(private string $planName)
    {
    }

    public function planName(): string
    {
        return $this->planName;
    }
}
```

---

#### 3. Caso especial

```php
class NullSubscription implements Subscription
{
    public function planName(): string
    {
        return 'No active subscription';
    }
}
```

---

#### 4. Repositório sempre retorna o contrato

```php
class SubscriptionRepository
{
    public function forUser(User $user): Subscription
    {
        $subscription = SubscriptionModel::where('user_id', $user->id)->first();

        if (!$subscription) {
            return new NullSubscription();
        }

        return new ActiveSubscription($subscription->plan_name);
    }
}
```

---

#### Uso sem condicionais

```php
$subscription = $repository->forUser($user);

echo $subscription->planName();
```

Não há `if`. O comportamento é polimórfico.

---

## Quando usar

* Quando há verificações repetidas de `null`
* Quando um estado especial possui comportamento previsível
* Para eliminar condicionais espalhadas
* Quando o caso especial é parte válida do domínio
* Para reduzir complexidade ciclomática

É especialmente útil quando o “caso especial” não representa erro, mas uma variação legítima de estado.

---

## Quando não usar

* Quando a ausência representa erro real
* Quando a falha deve interromper o fluxo (use exceção)
* Quando o caso especial não possui comportamento significativo
* Quando a complexidade adicional não compensa

Nem todo `null` precisa virar um objeto.

---

## Trade-offs envolvidos

**Vantagens:**

* Elimina condicionais repetidas
* Reduz complexidade ciclomática
* Favorece polimorfismo
* Melhora legibilidade
* Torna estados explícitos no modelo

**Desvantagens:**

* Aumenta número de classes
* Pode introduzir abstração desnecessária
* Pode mascarar situações que deveriam ser tratadas como erro
* Exige disciplina na modelagem

O padrão melhora clareza quando o caso especial é parte legítima do domínio, mas prejudica quando usado para esconder problemas.

---

## Resumo final

O Padrão de Caso Especial substitui verificações estruturais por comportamento orientado a objetos.

Em vez de perguntar:

```php
if ($object === null)
```

O sistema passa a trabalhar com:

* Um contrato claro
* Implementações distintas
* Polimorfismo

O resultado é um código mais expressivo, com menos ramificações condicionais e melhor modelagem de estados relevantes do domínio.
