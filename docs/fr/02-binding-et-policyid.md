# Chapitre 2 -- PolicyBinding, policyId et les payloads du cycle de vie

Une instance de policy est une autorisation specifique d un contrat de
policy donne, pour un compte donne, sous une liaison (binding)
determinee. Cette liaison est une struct `PolicyBinding` (definie dans
`src/PolicyManager.sol`) : `account`, `policy`, `policyConfig` (bytes
opaques interpretes par la policy), `validAfter`/`validUntil` (fenetre
de validite en secondes, zero desactivant la borne correspondante), et
`salt` (permet de creer plusieurs bindings distincts pour le meme
triplet compte/policy/config).

Le `policyId` de chaque instance est deterministe : c est le hash de
structure EIP-712 du binding, calcule par `getPolicyId`, avec
`policyConfig` hache separement (`keccak256(policyConfig)`) conformement
aux regles EIP-712 pour les types `bytes` dynamiques. Changer un seul
champ du binding, y compris le salt, produit un nouvel identifiant :
`policyId` nomme une instance d autorisation precise, pas la policy en
general.

Trois autres structs encadrent le reste du cycle de vie.
`ReplacePayload` regroupe l ancienne policy a desinstaller et le nouveau
binding a installer pour un remplacement atomique ; le code documente
explicitement en commentaire que `oldPolicyReplaceData` et
`newPolicyReplaceData` ne sont PAS couverts par la signature EIP-712 du
compte dans `replaceWithSignature` -- un relayer peut donc fournir ces
blobs librement, et toute policy qui s appuie dessus pour son
autorisation doit les valider elle-meme independamment (par exemple via
une signature d executeur specifique au contexte). `UninstallPayload`
unifie deux modes d adressage sous un seul point d entree : le mode
`policyId` (par `(policy, policyId)`, pour une instance deja installee)
et le mode `binding` (par le binding complet, qui permet aussi de
desactiver une instance jamais installee). Enfin `PolicyRecord` est
l enregistrement de stockage par `(policy, policyId)` : deux booleens
`installed`/`uninstalled` qui ne redeviennent jamais faux une fois vrais,
le compte associe, et les bornes de validite mises en cache.
