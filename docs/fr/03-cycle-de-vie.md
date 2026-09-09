# Chapitre 3 -- Le cycle de vie complet : install, execute, uninstall, replace

**Installation.** `install` (appel direct du compte) et
`installWithSignature` (signature ERC-6492, cote relayer) convergent
vers `_install`. L installation est idempotente : installer un
`policyId` deja installe est un no-op silencieux, sans re-emission
d evenement ni re-execution des hooks -- ce choix evite les courses
"premier installateur gagnant" et empeche qu une signature rejouee ne
redeclenche des effets de bord cote policy. `_install` rejette les
bindings deja expires (`validUntil` dans le passe) mais autorise
l installation anticipee avant `validAfter`, pour permettre un
pre-provisionnement. `installWithSignature` peut optionnellement
enchainer une execution si `executionData` est non vide ; la signature
du compte n autorise que le binding, jamais l executionData elle-meme --
la policy doit donc imperativement authentifier chaque execution de
maniere independante.

**Execution.** Le flux documente dans le README est : manager -> policy
-> manager -> account -> manager -> policy. Concretement,
`PolicyManager.execute` verifie que l instance est installee, delegue a
`_execute`, qui verifie que le `policyId` n est pas desinstalle, applique
la fenetre de validite (`_checkValidityWindow`, intervalle semi-ouvert
`[validAfter, validUntil)`), puis appelle `Policy.onExecute`, qui
retourne un `accountCallData` (le plan d appel) et un `postCallData`
optionnel. Si `accountCallData` est vide, le manager traite cela comme
un no-op total : pas d appel au compte, pas de `onPostExecute`, pas
d evenement `PolicyExecuted`. Sinon, le manager appelle le compte via
`Address.functionCall`, puis appelle `onPostExecute` sur la policy --
c est ce sandwich policy -> compte -> policy qui permet des
post-conditions fortes (verifier un delta de solde, un etat, une
reinitialisation d approbation) sans que le manager ait besoin de
comprendre la semantique specifique de la policy.

**Desinstallation.** `uninstall` est appelable par n importe quelle
adresse -- le manager ne restreint pas l appelant, il delegue entierement
l autorisation au hook `onUninstall` de la policy. La garantie unique du
manager : le compte peut toujours desinstaller ses propres instances
installees, meme si le hook de la policy revert (l "escape hatch") ;
dans ce cas le revert est avale et un evenement
`PolicyUninstallHookReverted` est emis a la place, mais pour tout
appelant autre que le compte, un revert du hook bloque bel et bien la
desinstallation. La desinstallation est permanente et irreversible pour
ce `policyId` precis -- reinstaller une policy equivalente exige un
nouveau salt.

**Remplacement.** `replace`/`replaceWithSignature` desinstallent une
instance et en installent une nouvelle de maniere atomique, avec sa
propre digest EIP-712 (`REPLACE_POLICY_TYPEHASH`) pour qu une signature
de remplacement ne puisse pas etre rejouee comme une simple installation.
Le remplacement est idempotent dans son etat final complet (ancienne
desinstallee, nouvelle installee et active), mais un etat partiel
(ancienne desinstallee sans que la nouvelle soit installee) n est pas
traite comme idempotent et fait revert -- le remplacement est
tout-ou-rien.
