# Chapitre 1 -- Presentation de account-policies

Ce depot definit un mecanisme onchain, independant du type de wallet,
pour installer sur un compte des modules d autorisation restreinte
appeles policies, puis executer des actions preparees par ces policies
via ce compte. L idee centrale : un compte (typiquement un smart
contract wallet) delegue sa capacite d execution a un contrat central,
le `PolicyManager`, qui lui-meme ne delegue qu une autorisation
specifique et limitee a chaque policy installee.

Le protocole est volontairement scinde en deux roles. `PolicyManager`
est un orchestrateur minimal : il suit les instances de policy
installees, verifie les autorisations d installation, applique les
transitions de cycle de vie et les invariants, et mediatise l execution
des policies sur les comptes utilisateur. Pour cela, il doit etre
proprietaire (owner) autorise a executer sur le smart contract wallet
de l utilisateur -- c est l ancre de confiance du systeme : le compte
delegue la capacite d execution au manager, qui delegue a son tour une
autorisation specifique a chaque policy.

`Policy` est une interface de hook minimale que les contrats de policy
implementent pour definir leur semantique d autorisation et construire
un calldata qui servira de plan d appel pour le wallet. Les policies
sont modulaires, extensibles, et connaissent l interface du wallet
(puisqu elles emettent des plans d appel) ; le `PolicyManager`, lui,
n a aucune connaissance prealable ou figee des policies specifiques.

L objectif declare du design : garder le manager stable et generique,
tout en laissant les policies exprimer la logique specifique a chaque
application -- ce qui est autorise, sous quelles conditions, et
comment l executer en toute securite. Cela ouvre des usages comme
l automatisation d actions recurrentes, l execution deleguee a un
tiers (relayer/executor) autorisee par signature, des actions
conditionnelles, ou des actions budgetisees avec des limites
periodiques -- sans jamais donner un controle total et illimite du
compte.
