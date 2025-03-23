---
title: "Projets"
date: 2022-06-13T20:55:37+01:00
draft: false

showDate: false
showDateOnlyInArticle: false
showDateUpdated: false
showHeadingAnchors: false
showPagination: false
showReadingTime: false
showTableOfContents: true
showTaxonomies: false
showWordCount: false
showSummary: false
sharingLinks: false
showEdit: false
layoutBackgroundHeaderSpace: false
#groupByYear : false
---

Let me cook.

<table>
    <thead>
        <tr>
            <th>Logo</th>
            <th>Titre</th>
            <th>Description</th>
            <th>Références</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img class="customEntitityAlbum" style="background-color:transparent" src="homelab.fr.png"/></td>
            <td>
              Homelab
              {{< badge >}}
                Bientôt disponible
              {{< /badge >}}
            </td>
            <td>Mon homelab où j'exécute mes propres services.</td>
            <td><a target="_blank" href=""></a></td>
        </tr>
         <tr>
            <td><img class="customEntitityAlbum" style="background-color:transparent" src="hugo.fr.png"/></td>
            <td>
              Mon site web
              {{< badge >}}
                Actif
              {{< /badge >}}
            </td>
            <td>Mon site web statique simple, réalisé avec <a target="_blank" href="https://gohugo.io/">Hugo</a> & <a target="_blank" href="https://blowfish.page/">Blowfish</a>.</td>
            <td><a target="_blank" href="https://github.com/justin-de-sio/my-blog">Dépôt Github</a></td>
        </tr>
        <tr>
            <td><img class="customEntitityAlbum" style="background-color:transparent" src="cicd.fr.png"/></td>
            <td>
              JSTAcademy
              {{< badge >}}
                Archivé
              {{< /badge >}}
            </td>
            <td>
                <p>JST Academy est une plateforme interactive de formation en ligne proposant des cours structurés avec des leçons et des quiz. Elle utilise des technologies modernes avec <strong>Angular</strong> pour le frontend et <strong>Java Spring Boot</strong> pour le backend.</p>
                <p>Son infrastructure repose sur deux <strong>pipelines CI/CD</strong> automatisés et gérés via GitLab :</p>
                <ul>
                <li>Le premier pipeline construit les images Docker, exécute les tests d'intégration et déploie sur <strong>AWS EC2</strong> grâce à Ansible.</li>
                <li>Le second pipeline provisionne et gère l'infrastructure cloud sur <strong>AWS EC2</strong> à l'aide de <strong>Terraform</strong>.</li>
                </ul>
            </td>
                <td><a target="_blank" href="https://gitlab.com/jstacademy">Dépôt GitLab</a></td>
        </tr>

</tbody>

</table>
