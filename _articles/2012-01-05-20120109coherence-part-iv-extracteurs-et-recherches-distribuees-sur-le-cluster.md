---
ID: 408
post_title: 'Coherence Part IV : extracteurs et recherches distribuées sur le cluster'
author: team-bourgain-tinon
post_date: 2012-01-05 09:30:00
post_excerpt: "<p>Cet article est la suite du précédent, qui montrait comment réaliser des recherches distribuées sur le cluster avec des filtres. Celui ci va plus loin et expose le mécanisme d'extraction des données à présenter au filtre à partir des entrées du cache.</p>"
layout: post
permalink: http://blog.zenika-offres.com/?p=408
published: true
slide_template:
  - ""
---
<p>Cet article est la suite du précédent, qui montrait comment réaliser des recherches distribuées sur le cluster avec des filtres. Celui ci va plus loin et expose le mécanisme d'extraction des données à présenter au filtre à partir des entrées du cache.</p>
<!--more-->
<h3>Préparatifs&nbsp;: adaptation du modèle</h3> <p>Pour continuer à approcher d'une application adaptée à un datagrid, nous devons modifier le modèle. Cela nous permettra aussi de vous présenter toujours plus de features sympas de Coherence ;)</p> <p>Le premier défaut du modèle actuel réside dans l'accès aux <strong>Visit</strong>s qui requiert de récupérer un <strong>Owner</strong> puis d'itérer sur les <strong>Pet</strong>s. Cette structure rend donc la manipulation des <strong>Visit</strong>s dans le datagrid peu performante et pratique.</p> <p>Pour améliorer les choses, nous vous proposons de stocker les <strong>Visit</strong>s dans un cache séparé. Ceci impose une première modification du modèle pour remplacer la référence directe entre <strong>Pet</strong> et <strong>Visit</strong> par une référence sur les IDs des entités correspondantes. Sans cette modification, l'insertion d'une <strong>Visit</strong> dans le cache associé sérialiserait à la fois la <strong>Visit</strong> et tout le graphe d'objets concerné. De plus, le cache des <strong>Owner</strong>s contiendrait toujours les <strong>Visit</strong>s.</p> <p>Nous pouvons en profiter pour réaliser une seconde modification qui nous simplifiera la vie. L'application PetClinic n'utilise le lien entre <strong>Pet</strong> et <strong>Visit</strong> que dans le sens Pet -&gt; Visit. Nous allons donc retirer toute référence aux <strong>Visit</strong>s dans <strong>Pet</strong>.</p> <p>Nous avons fait toutes ces modifications fastidieuses pour vous (qui aime corriger des JSPs ?), comme d'habitude, le projet est disponible&nbsp;: - GitHub</p> <pre class="bash code bash" style="font-family:inherit">git clone git:<span style="color: #000000; font-weight: bold;">//</span>github.com<span style="color: #000000; font-weight: bold;">/</span>obourgain<span style="color: #000000; font-weight: bold;">/</span>petclinic-coherence.git git checkout article4-start</pre> <p>- <a href="http://blog.zenika.com/public/Billet_294/petclinic-coherence-article4-start.zip">zip</a></p> <h3>ValueExtractor</h3> <p>Dans l'article précédent nous avions requêté le cache avec une opération de type map/reduce grâce à un filtre avec le nom de la méthode à appeler par réflexion, mais Coherence peut aussi répondre à des problématiques de recherche plus poussées et de façon plus robuste grâce aux extracteurs.</p> <p>L'interface <strong>ValueExtractor</strong> de Coherence permet de récupérer une valeur particulière pour chaque objet du cache. Il est possible de récupérer un attribut, appeler une méthode ou retourner n'importe quelle valeur qui vous passe par la tête. Les extracteurs peuvent êtres combinés aux filtres pour requêter le cache selon des critères complexes&nbsp;: le filtre s'applique sur la valeur retournée par l'extracteur.</p> <p>Le filtre de l'article précédent utilise d'ailleurs implicitement un extracteur pour récupérer le résultat de la méthode donnée par reflexion. On pourrait modifier le <strong>LikeFilter</strong> de l'article précédent pour utiliser explicitement un <strong>ReflexionExtractor</strong>, le fonctionnement resterait le même&nbsp;:</p> <pre class="java code java" style="font-family:inherit">Filter lastNameFilter = <span style="color: #7F0055; font-weight: bold;">new</span> LikeFilter<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">new</span> ReflectionExtractor<span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;getLastName&quot;</span><span style="color: #000000;">&#41;</span>, lastName + <span style="color: #888888;">&quot;%&quot;</span>, <span style="color: #888888;">'<span style="color: #000099; font-weight: bold;">\</span>'</span>, <span style="color: #7F0055; font-weight: bold;">true</span><span style="color: #000000;">&#41;</span>;</pre> <p>Les extracteurs apportent beaucoup de souplesse dans les possibilités de requêtage. Coherence propose quelques autres extracteurs mais dans la plupart des cas il faudra une nouvelle implémentation de l'interface pour adapter les requêtes au modèle&nbsp;: query = code&nbsp;!</p> <h3>Back to business</h3> <p>La méthode <strong>loadPet()</strong>, qui itère sur le cache des <strong>Owner</strong>s, va pouvoir tirer parti des extracteurs pour paralléliser et distribuer le traitement. Nous allons utiliser un extracteur qui retourne une <strong>Collection</strong> des IDs de tous les <strong>Pet</strong>s pour chaque <strong>Owner</strong>. Les IDs extraient vous ensuite se voir appliquer un <strong>ContainsFilter</strong> pour chercher l'ID demandé dans la Collection d'IDs. Nous récupérons la Collection d'Entry pour lesquelles le filtre a retourné true, c'est à dire une Collection qui contient le seul <strong>Owner</strong> lié au <strong>Pet</strong> recherché. Il faut maintenant parcourir les <strong>Pet</strong>s de ce <strong>Owner</strong> pour trouver le bon. Voici l'extracteur&nbsp;:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">class</span> PetIdsExtractor <span style="color: #7F0055; font-weight: bold;">implements</span> ValueExtractor <span style="color: #000000;">&#123;</span> 	<span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #000000;">Object</span> extract<span style="color: #000000;">&#40;</span><span style="color: #000000;">Object</span> o<span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> 		<span style="color: #000000;">Owner</span> owner = <span style="color: #000000;">&#40;</span><span style="color: #000000;">Owner</span><span style="color: #000000;">&#41;</span> o; 		List<span style="color: #000000;">&lt;</span>Integer<span style="color: #000000;">&gt;</span> extractedPetIds = <span style="color: #7F0055; font-weight: bold;">new</span> ArrayList<span style="color: #000000;">&lt;</span>Integer<span style="color: #000000;">&gt;</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; 		<span style="color: #7F0055;font-weight: bold;">for</span> <span style="color: #000000;">&#40;</span>Pet pet : owner.<span style="color: #000000;">getPets</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span><span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> 			extractedPetIds.<span style="color: #000000;">add</span><span style="color: #000000;">&#40;</span>pet.<span style="color: #000000;">getId</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span><span style="color: #000000;">&#41;</span>; 		<span style="color: #000000;">&#125;</span> 		<span style="color: #7F0055; font-weight: bold;">return</span> extractedPetIds; 	<span style="color: #000000;">&#125;</span> <span style="color: #000000;">&#125;</span></pre> <p>et son utilisation dans le filtre&nbsp;:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055; font-weight: bold;">public</span> Pet loadPet<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">int</span> id<span style="color: #000000;">&#41;</span> <span style="color: #7F0055; font-weight: bold;">throws</span> DataAccessException <span style="color: #000000;">&#123;</span> 	ContainsFilter filter = <span style="color: #7F0055; font-weight: bold;">new</span> ContainsFilter<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">new</span> PetIdsExtractor<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>, id<span style="color: #000000;">&#41;</span>; &nbsp; 	<span style="color: #808080; font-style: italic;">// there should be at most one owner matching the filter</span> 	Set<span style="color: #000000;">&lt;</span>Entry<span style="color: #000000;">&lt;</span>Integer, Owner<span style="color: #000000;">&gt;&gt;</span> entries = getOwnersCache<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span
>.<span style="color: #000000;">entrySet</span><span style="color: #000000;">&#40;</span>filter<span style="color: #000000;">&#41;</span>; &nbsp; 	<span style="color: #7F0055;font-weight: bold;">if</span> <span style="color: #000000;">&#40;</span>entries.<span style="color: #000000;">isEmpty</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span><span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> 		<span style="color: #7F0055; font-weight: bold;">return</span> <span style="color: #7F0055; font-weight: bold;">null</span>; 	<span style="color: #000000;">&#125;</span> &nbsp; 	Entry<span style="color: #000000;">&lt;</span>Integer, Owner<span style="color: #000000;">&gt;</span> entry = entries.<span style="color: #000000;">iterator</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>.<span style="color: #000000;">next</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; 	<span style="color: #000000;">Owner</span> owner = entry.<span style="color: #000000;">getValue</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; 	<span style="color: #7F0055;font-weight: bold;">for</span> <span style="color: #000000;">&#40;</span>Pet pet : owner.<span style="color: #000000;">getPets</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span><span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> 		<span style="color: #7F0055;font-weight: bold;">if</span> <span style="color: #000000;">&#40;</span>pet.<span style="color: #000000;">getId</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>.<span style="color: #000000;">equals</span><span style="color: #000000;">&#40;</span>id<span style="color: #000000;">&#41;</span><span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> 			<span style="color: #7F0055; font-weight: bold;">return</span> pet; 		<span style="color: #000000;">&#125;</span> 	<span style="color: #000000;">&#125;</span> 	<span style="color: #7F0055; font-weight: bold;">return</span> <span style="color: #7F0055; font-weight: bold;">null</span>; <span style="color: #000000;">&#125;</span></pre> <p>Ce code est plus long qu'avant, mais c'est le prix de l'efficacité&nbsp;: maintenant le traitement est distribué sur le cluster. Rassurez-vous, les recherches dans la datagrid peuvent être beaucoup plus courtes, ici c'est la recherche dans la Collection qui est verbeuse.<br />
De manière analogue à une base de données relationelle, plus le cache contient d'entrées, plus l'opération devient coûteuse en CPU puisque le cache est parcouru et chaque entrée doit être de-sérialisée avant d'appliquer l'extracteur, même si le traitement est distribué il peut y avoir une augmentation du temps de traitement. Les solutions pour pallier ce problème sont d'utiliser un index, ce sera le sujet du prochain article, ou un algorithme de sérialisation plus performant.</p> <p>Avec les modifications du modèle, nous devons ajouter une méthode pour récupérer les <strong>Visit</strong>s d'un <strong>Pet</strong>. Cette méthode peut aussi bénéficier de la puissance d'un extracteur&nbsp;:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">class</span> PetIdFromVisitExtractor <span style="color: #7F0055; font-weight: bold;">implements</span> ValueExtractor <span style="color: #000000;">&#123;</span> 	<span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #000000;">Object</span> extract<span style="color: #000000;">&#40;</span><span style="color: #000000;">Object</span> o<span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> 		Visit visit = <span style="color: #000000;">&#40;</span>Visit<span style="color: #000000;">&#41;</span> o; 		<span style="color: #7F0055; font-weight: bold;">return</span> visit.<span style="color: #000000;">getPetId</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; 	<span style="color: #000000;">&#125;</span> <span style="color: #000000;">&#125;</span></pre> <p>Avec cet extracteur, nous pouvons récupérer l'ensemble des <strong>Visit</strong>s comme suit&nbsp;:</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055; font-weight: bold;">public</span> Set<span style="color: #000000;">&lt;</span>Visit<span style="color: #000000;">&gt;</span> loadVisitsForPet<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">int</span> petId<span style="color: #000000;">&#41;</span> <span style="color: #7F0055; font-weight: bold;">throws</span> DataAccessException <span style="color: #000000;">&#123;</span> 	Filter filter = <span style="color: #7F0055; font-weight: bold;">new</span> EqualsFilter<span style="color: #000000;">&#40;</span><span style="color: #7F0055; font-weight: bold;">new</span> PetIdFromVisitExtractor<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>, petId<span style="color: #000000;">&#41;</span>; 	Set<span style="color: #000000;">&lt;</span>Entry<span style="color: #000000;">&lt;</span>Integer, Visit<span style="color: #000000;">&gt;&gt;</span> visits = getVisitsCache<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>.<span style="color: #000000;">entrySet</span><span style="color: #000000;">&#40;</span>filter<span style="color: #000000;">&#41;</span>; 	Set<span style="color: #000000;">&lt;</span>Visit<span style="color: #000000;">&gt;</span> result = <span style="color: #7F0055; font-weight: bold;">new</span> HashSet<span style="color: #000000;">&lt;</span>Visit<span style="color: #000000;">&gt;</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>; 	<span style="color: #7F0055;font-weight: bold;">for</span> <span style="color: #000000;">&#40;</span>Entry<span style="color: #000000;">&lt;</span>Integer, Visit<span style="color: #000000;">&gt;</span> entry : visits<span style="color: #000000;">&#41;</span> <span style="color: #000000;">&#123;</span> 		result.<span style="color: #000000;">add</span><span style="color: #000000;">&#40;</span>entry.<span style="color: #000000;">getValue</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span><span style="color: #000000;">&#41;</span>; 	<span style="color: #000000;">&#125;</span> 	<span style="color: #7F0055; font-weight: bold;">return</span> result; <span style="color: #000000;">&#125;</span></pre> <p>Dans ce cas nous retournons l'ensemble des résultats récupérés via le filtre. Coherence ne propose pas de méthode du type <strong>values()</strong> avec un filtre, il faut donc itérer sur l'entrySet pour les récupérer.</p> <p>Le projet est téléchargeable&nbsp;: Sur GitHub</p> <pre class="bash code bash" style="font-family:inherit">git clone git:<span style="color: #000000; font-weight: bold;">//</span>github.com<span style="color: #000000; font-weight: bold;">/</span>obourgain<span style="color: #000000; font-weight: bold;">/</span>petclinic-coherence.git git checkout article4-start</pre> <p>En <a href="http://blog.zenika.com/public/Billet_294/petclinic-coherence-article4-end.zip">zip</a></p> <h3>Conclusion</h3> <p>Les extracteurs de Coherence sont puissants et combinés aux filtres que nous avons présentés dans l'article précédent, vous êtes maintenant capable de récupérer de façon efficace et distribuée les données dans les caches. Il est toujours nécessaire de de-sérialiser chaque entrée puis appliquer l'extracteur et le filtre dessus, ce qui peut être problématique sur des caches contenant beaucoup d'entrées ou avec des contraintes de vitesse. Le prochain article montrera comment ajouter des index dans le caches afin d'accélérer les recherches.</p> <p>Index des articles de la série Coherence&nbsp;:</p> <ul> <li><a href="index.php?post/2011/08/12/Coherence-PetCinic">Introduction à Coherence: Part I</a></li> <li><a href="index.php?post/2011/08/25/Introduction-à-Coherence%3A-Part-II">Introduction à Coherence: Part II</a></li> <li><a href="index.php?post/2011/10/07/Introduction-à-Coherence%3A-Part-III">Coherence Part III&nbsp;: Filtres</a></li> <li><a href="index.php?post/2012/01/09/Coherence-Part-IV-%3A-extracteurs-et-recherches-distribuées-sur-le-cluster">Coherence Part IV&nbsp;: extracteurs et recherches distribuées sur le cluster</a></li> <li><a href="index.php?post/2012/01/03/Coherence-Part-IV-%3A-optimisations-des-requêtes-avec-des-index6">Coherence Part V&nbsp;: optimisations des requêtes avec des index</a></li> <li><a href="index.php?post/2012/01/06/Coherence-Part-VI-%3A-traitement-de-données-distribuées%2C-in-place-processing4">Coherence Part VI&nbsp;: traitement de données distribuées, concurrence et in-place processing</a></li> </ul>