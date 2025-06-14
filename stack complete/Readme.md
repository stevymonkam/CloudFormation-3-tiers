# 📦 AWS CloudFormation 3-Tier Architecture with Nested Stacks

si tu veux deployer l'infra que tu as ici en bas tu prend juste cette stack 
tu importe dans cloudFormation tu lance sa fonctione la stack du bas ne fonctione pas totalement il ya des petit bug mais si tu veux corriger la stack du bas tu peut te refere a cette stack et comprendre et corriger celle du bas

pour que sa marche il faut creer une key pair de nom : devops-stevy

---

NB : VOICI COMMENT LA PIPELINE DOIT ETRE DEFINIE EN ENVIRONEMENT DE PROD : SUPPOSONS QUE TU TRAVAILLE SUR UNE FONCTIONALITER 
EN LOCAL ET QUE TU AS FINIS AVEC UNE FONCTIONALITER QUAND TU VAS FAIRE LA PUSH SUR GIT SUR TA BRANCHE FEATUR AVANT SA TU COPIE DABOR TOI MEME TON YAML SUR S3 DE DEV (ON PEUT FAIRE TOUS CE KON VEUX EN DEV) ENSUITE LA PIELINE EST LANCER ET RECUPERE LE CODE SUR S3 ET FAIT L'APPLY SI TOUS VAS BIEN TU FAIT LA MERGE SUR LA BRANCHE DEV QUI EST LIE A UNE AUTRE PIPELINE SI LA MERGE EST APROUVER CET A DIRE LE CODE DU YAML EST APROUVER LA PIPELINE COPIE DABORD LES NOUVEL MODIF DU CODE SUR S3 DE DEVELOPPEMENT QUI EST DIFFERENT DE CELUI DE TEST ET APRES LANCE L'APPLY
