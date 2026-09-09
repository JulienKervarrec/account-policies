# Chapitre 4 -- L interface Policy et la repartition des responsabilites

`src/policies/Policy.sol` est un contrat abstrait qui definit cinq hooks
externes, tous restreints au seul `PolicyManager` (`onlyPolicyManager`) :
`onInstall`, `onExecute`, `onPostExecute`, `onUninstall`, et `onReplace`.
Chacun delegue a une fonction interne `_xxx` que les policies concretes
doivent implementer (ou dont elles heritent d une implementation par
defaut). `onReplace` illustre bien le design : selon le `ReplaceRole`
passe (`OldPolicy` ou `NewPolicy`), il route vers
`_onUninstallForReplace` ou `_onInstallForReplace`, dont les
implementations par defaut delegent respectivement vers `_onUninstall`
et `_onInstall` -- une policy qui ne se soucie pas de distinguer un
remplacement d un cycle de vie standard n a rien a surcharger.

Le README trace une frontiere de responsabilite explicite entre le
manager et les policies. Le manager s occupe de : calculer le `policyId`
deterministe, valider les signatures/appels du compte pour les
installations (capable ERC-6492), transmettre `policyConfig` a la policy
a l installation, rejeter les fenetres de validite expirees ou
impossibles a l installation et les faire respecter a l execution,
verifier que l adresse de la policy a du code deploye et persistant,
maintenir l etat de vie des instances, imposer les desactivations
definitives, et garantir que le compte peut toujours desinstaller. Les
policies, elles, sont seules responsables de : l autorisation
d execution (qui peut executer et sous quelles conditions), le decodage
et la validation de `policyConfig`/`executionData`, la protection contre
le rejeu et la discipline de nonce, les limites et invariants propres a
la policy (budgets, bornes de slippage, seuils), tout etat specifique,
les regles optionnelles de desinstallation par un tiers, et la
validation post-appel optionnelle.

Un point de securite critique documente dans le README : le manager
appelle `execute()` de maniere permissionless, et `PolicyManager.execute`
peut etre invoque par n importe qui -- c est entierement a
`onExecute` de chaque policy de valider l appelant (par exemple via une
signature d executeur). Une policy qui ne valide pas l appelant dans
`onExecute` permettrait a n importe qui de declencher une execution.
Le README precise egalement que les contrats de policy ne doivent
jamais etre deployes derriere un proxy amendable (upgradeable) : puisque
le `PolicyManager` est owner du wallet et transmet aveuglement le
calldata retourne par la policy, un admin de proxy malveillant pourrait
changer la logique d une policy deja installee pour lui faire retourner
un calldata arbitraire -- retirer des owners, en ajouter, transferer des
fonds.
