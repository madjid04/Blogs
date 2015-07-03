---
ID: 234
post_title: 'Concours &#8211; Développer une application web en Wicket'
author: Carl Azoury
post_date: 2009-03-10 20:30:00
post_excerpt: "<p>Développer des applications web peut être plus moins long et difficile en fonction du framework web utilisé. Wicket étonne par sa simplicité, sa rapidité de développement et l'élégance du modèle de programmation. Il s'agit d'un framework web de type composant entièrement en Java pour la partie serveur et en xHTML / CSS pour la partie cliente. Wicket possède également un moteur Ajax fortement intégré au framework permettant de bénéficier de certaines fonctionnalités très facilement. Pour bien développer en Wicket&nbsp;? Il suffit de maîtriser le langage Java et les concepts objets.</p>"
layout: post
permalink: http://blog.zenika-offres.com/?p=234
published: true
---
<p>Développer des applications web peut être plus moins long et difficile en fonction du framework web utilisé. Wicket étonne par sa simplicité, sa rapidité de développement et l'élégance du modèle de programmation. Il s'agit d'un framework web de type composant entièrement en Java pour la partie serveur et en xHTML / CSS pour la partie cliente. Wicket possède également un moteur Ajax fortement intégré au framework permettant de bénéficier de certaines fonctionnalités très facilement. Pour bien développer en Wicket&nbsp;? Il suffit de maîtriser le langage Java et les concepts objets.</p>
<!--more-->
<p>Lors de la <a href="http://www.parisjug.org/xwiki/bin/view/Meeting/20090310" hreflang="fr">soirée web du Paris JUG</a> du 10 mars 2009, nous avons développé une mini application web de gestion de contacts. Certes il ne s'agit pas d'un cas réel d'entreprise, mais cette application permet déjà d'illustrer différentes fonctionnalités sommes toutes assez variées.</p> <p>La diversité des "use case" ainsi développés permet d'initier un comparatif entre frameworks web. Les critères importants de ce comparatif sont le temps de réalisation, le nombre de lignes de code produites ainsi que le nombre de lignes de configuration (XML ou annotations).</p> <p>Nous fournissons, via ce blog, un Starter Kit sous forme de projet Eclipse contenant les fichiers HTML ainsi que les ressources (images / CSS). Nous fournissons aussi la solution, sous forme de projet Eclipse également, telle que développée lors du Paris JUG.</p> <p>Le concours consiste à développer la même application avec le framework web de votre choix en utilisant le "Starter Kit" mis à disposition. N'hésitez pas à nous poster vos résultats via les commentaires et nous ferons un résumé comparatif avec ceux ci que nous publierons pour le prochain JUG.</p> <p>Pour information les fonctionnalités de l'application fournie en solution sont les suivantes&nbsp;:</p> <ul> <li>La navigation entre pages</li> <li>L'organisation d'une structure commune des pages (Type Tiles / Sitemesh)</li> <li>La désactivation du lien correspondant à la page courante</li> <li>L'édition d'un contact</li> <li>La création d'un contact</li> <li>Lister les contacts</li> <li>L'ajout rapide d'un contact dans une liste</li> <li>Le refresh de la date courante via appels Ajax</li> <li>L'édition "in place" d'un libellé sans passer par un écran d'édition</li> <li>L'ajout de la validation (Nom et prénom obligatoires, contrôle du format date, contrôle du type email)</li> <li>Utilisation d'un date picker</li> <li>Synchronisation du format du DatePicker avec le format utilisé par le convertisseur</li> <li>Ajout des mêmes contrôles de validation côté client, en Javascript</li> <li>Gestion de la problématique du refresh afin d'éviter la double soumission</li> <li>Non duplication du code du formulaire... le même composant doit être utilisé pour la page d'édition de contacts et de liste des contacts</li> <li>Le drag &amp; drop depuis la liste vers le formulaire d'édition</li> <li>L'affichage de message d'erreurs</li> <li>La réutilisation des mêmes messages d'erreurs en validation serveur et javascript</li> <li>Le tri de la liste des contacts par nom et par prénom</li> </ul> <p>Le résultat du développement avec Wicket est le suivant&nbsp;:</p> <ul> <li>12 classes (classes internes incluses et la classe Contact exclue)</li> <li>163 lignes de codes</li> <li>1 seule configuration XML (Le filtre dans le web.xml)</li> <li>0 lignes de javascript codées</li> <li>Temps de développement &lt; 1 heure</li> </ul> <p>Pour information, la classe Contact à elle seule représente 33 lignes de code, soit 17% de l'application.</p> <p>Bon courage et un livre Wicket in Action sera offert par tirage au sort à l'un des participants de ce concours&nbsp;!</p>