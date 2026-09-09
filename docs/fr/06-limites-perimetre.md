# Chapitre 6 -- Limites et perimetre de ce parcours

Ce parcours couvre en profondeur le coeur generique et reutilisable du
protocole : `src/PolicyManager.sol` (types, cycle de vie complet,
frontieres de confiance), `src/policies/Policy.sol` (interface de hook)
et la paire `SingleExecutorPolicy.sol` /
`SingleExecutorAuthorizedPolicy.sol` (le pattern d autorisation par
executeur signe, partage par toutes les policies concretes du depot).

Sont volontairement laisses hors champ le detail d implementation des
policies metier concretes situees dans `src/policies/`
(`MorphoLendPolicy.sol`, `MorphoLoanProtectionPolicy.sol`,
`MorphoWethLoanProtectionPolicy.sol`, `TransferSettingsPolicy.sol`, et le
sous-dossier `accounting/`) : chacune encode une logique specifique a
Morpho ou a des regles de transfert de token qui meriterait un parcours
dedie plutot qu un resume superficiel ici. Sont egalement hors champ
`PublicERC6492Validator.sol` (le validateur de signature externe
mentionne comme dependance mais non detaille), les interfaces du dossier
`src/interfaces/` (`IWETH.sol`, `interfaces/morpho/`), les scripts de
deploiement (`script/`), la suite de tests Foundry (`forge test
--offline`, non executee dans le cadre de ce parcours documentaire), et
le contenu detaille des quatre rapports d audit Spearbit/Cantina lies
dans le README.

L objectif reste le meme que pour les parcours precedents de cette
bibliotheque : comprendre precisement comment un orchestrateur
generique et minimal peut deleguer une autorisation d execution
restreinte a des modules extensibles, avec des frontieres de confiance
et des invariants de cycle de vie explicites, sans pretendre couvrir
l integralite des policies concretes deployees par Base.
