# Revue documentaire Discord — 19 octobre 2026

## Portée et résultat

Comparaison de terms.html et privacy.html avec :
- https://support-dev.discord.com/hc/articles/8562894815383-Discord-Developer-Terms-of-Service
- https://support-dev.discord.com/hc/articles/8563934450327-Discord-Developer-Policy

Les deux pages Discord consultées indiquent une mise à jour au 6 juin 2024 et une entrée en vigueur au 8 juillet 2024. Cette revue documentaire ne certifie ni la conformité juridique globale ni les pratiques de l'application : aucun audit du code RPG, de la production ou du Developer Portal n'a été réalisé.

Les textes existants couvraient déjà la gratuité, les actifs virtuels sans valeur monétaire, l'âge Discord, les données de jeu, un contact et des droits RGPD. Les changements clarifient la priorité des règles Discord, les signalements, les autorisations, les limites d'usage des données, les destinataires et les circonstances de suppression. Les deux langues sont mises à jour. Le résumé des données n'affirme plus « rien de plus » malgré la liste détaillée. Les polices distantes sont retirées et les liens de confidentialité corrigés.

## Vérifications avant fusion et publication

Ces éléments sont à vérifier, et ne sont pas des constats de conformité :
- [ ] Tester le contact Discord legosi_ et le relais privé par un modérateur, y compris pour une personne ayant quitté le serveur. Prévoir un contact indépendant du serveur si nécessaire.
- [ ] Vérifier les données réellement reçues, persistées et journalisées : OAuth, rôles, avatar, IP, journaux du proxy/hébergeur, erreurs et modération. Corriger la politique si l'inventaire diffère.
- [ ] Confirmer le chiffrement au repos de la base ET des sauvegardes (Developer Terms §5(c)), les accès et la protection des secrets. Ne pas considérer cette exigence comme satisfaite par le texte.
- [ ] Tester la rectification et la suppression sur base active, journaux, exports et sauvegardes. Fixer une durée maximale précise de rotation des sauvegardes et empêcher la réapparition de données après restauration.
- [ ] Vérifier la purge effective des logs après 14 jours déjà annoncée dans la politique et définir le critère d'inactivité justifiant l'effacement des comptes devenus inutiles.
- [ ] Identifier les prestataires, lieux de traitement et transferts internationaux ; compléter les informations et garanties applicables. Vérifier les engagements des prestataires.
- [ ] Documenter la procédure d'incident : notification rapide à Discord, information des personnes selon la loi applicable, remédiation et coopération (Developer Terms §5(c)).
- [ ] Vérifier que les URL publiques figurent dans le Developer Portal et restent accessibles depuis le bot ET l'Activité ; tester les permissions et les choix de blocage/retrait.
- [ ] Confirmer l'identité et les coordonnées adéquates de l'éditeur, la base légale RGPD et le choix du droit français déjà présents ; préserver les droits impératifs des utilisateurs et identifier l'autorité compétente.
- [ ] Confirmer les pratiques déjà annoncées : aucun paiement, collecte de messages, stockage persistant du jeton OAuth ou suivi publicitaire ; vérifier les licences des ressources et la classification d'âge applicable.
- [ ] Si des fonctions payantes sont ajoutées, refaire la revue des règles de monétisation Discord et de Premium Apps avant lancement.

Les engagements ajoutés doivent être effectivement appliqués avant publication. La branche principale publie GitHub Pages : cette proposition reste en pull request jusqu'à validation de ces points.
