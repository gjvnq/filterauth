# Examples

Syntax and language notes:

  * Keywords (like Clojure): ```:my-keyword```
  * Multilangual deny messages.
  * Parameters that don't appear match all. (see ```admin-is-god``` rule)
  * Configurable parameters (```params```) that are separate from the rules themselves.

```
rule separation-of-concerns-payment(actor: User, verb=:approve, object: Payment):
	deny-msg[:en] = "The person who creates the the person who approves the payment must not be the same"
	deny-msg[:pt] = "A pessoa que cria e a pessoa que aprova um pagamento não podem ser a mesma"
	priority = 10
	if actor.id == object.creator-id:
		return deny
	return continue

rule large-payments-require-manager(actor: User, verb=:approve, payment: Payment):
	deny-msg[:en] = "Only managers can approve payments larger than {params.large-payment-threshold}"
	deny-msg[:pt] = "Apenas gerentes podem aprovar pagamentos maiores que {params.large-payment-threshold}"
	priority = 10
	if :manager in actor.roles and payment.amount > params.large-payment-threshold:
		return deny
	return continue

rule admin-is-god(actor: User):
	deny-msg[:en] = "You are not and admin"
	deny-msg[:pt] = "Você não é um administrador"
	priority = 0
	if actor.is-admin:
		return allow
	return continue
```

How a query is answered:

  1. Sort applicable rules by priority.
  2. If no rules apply, deny the query.
  3. Group applicable rules by priotirty.
  4. For each priority grouping in ascending order (lowest to highest):
    1. If any rule returns deny, deny the query.
    2. If at least one rule returns allow, allow the query.
    3. Continue to the next priority group.