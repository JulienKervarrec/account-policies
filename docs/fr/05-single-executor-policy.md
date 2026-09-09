# Chapitre 5 -- Le pattern SingleExecutorPolicy et ses variantes

`src/policies/SingleExecutorPolicy.sol` est une base abstraite pour les
policies qui s appuient sur une seule adresse executeur pour leur
autorisation. Elle possede l encodage ABI canonique partage par toutes
les sous-classes : `policyConfig = abi.encode(SingleExecutorConfig{executor},
bytes policySpecificConfig)` et
`executionData = abi.encode(SingleExecutorExecutionData{nonce, deadline,
signature}, bytes actionData)`.

Ce contrat est deliberement calldata-heavy plutot que storage-heavy (une
des deux strategies de gestion de config documentees au chapitre 4) : il
ne stocke qu un hash du config (`_configHashByPolicyId`) et exige a
chaque execution le preimage complet du config, verifie par
`_requireConfigHash`. Il gere aussi le remplacement de nonces
(`_usedNonces` par `policyId`), avec une fonction `cancelNonces` que seul
l executeur configure peut appeler -- volontairement non soumise a
`whenNotPaused`, car une revocation de securite doit toujours fonctionner
meme si la policy est en pause. Trois digests EIP-712 distincts
structurent les signatures : `EXECUTION_TYPEHASH` (execution),
`SINGLE_EXECUTOR_UNINSTALL_TYPEHASH` (desinstallation par un tiers), et
l ensemble herite d `AccessControl` (role `PAUSER_ROLE`) et de
`Pausable` d OpenZeppelin.

`SingleExecutorAuthorizedPolicy.sol` implemente le pattern
"toujours-autorise" au-dessus de cette base : chaque execution doit
imperativement etre signee par l executeur configure (pas d execution
sans signature valide). `_onInstall` exige un executeur non nul.
`_onUninstall` distingue trois cas : le compte lui-meme peut toujours
desinstaller sans fournir de config (l escape hatch du chapitre 3) ; une
desactivation pre-installation exige une signature d executeur sur un
digest specifique ; une desinstallation post-installation par un tiers
exige a la fois le preimage du config stocke et une signature
d executeur fraiche. `_onExecute` compose ces briques : verifie que la
policy n est pas en pause, verifie le hash du config, decode
l enveloppe d execution, puis appelle
`_validateAndConsumeExecutionIntent` -- qui consomme le nonce (rejeu
impossible), verifie l expiration eventuelle de la signature, et
verifie la signature de l executeur sur un digest qui lie ensemble le
`policyId`, le compte, le hash du config, et le hash de l action --
avant de deleguer enfin a `_onSingleExecutorExecute`, le point
d extension que chaque policy concrete (par exemple les policies Morpho
du dossier `src/policies/`) doit implementer pour sa logique metier
propre.
