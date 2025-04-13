---
title: "Projects"
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
            <th>Title</th>
            <th>Description</th>
            <th>References</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><img class="customEntitityAlbum" style="background-color:transparent" src="homelab.png"/></td>
            <td>
              Homelab
              {{< badge >}}
                Active
              {{< /badge >}}
            </td>
            <td>My homelab where I host my services.</td>
            <td><a target="_blank" href="https://github.com/Justin-De-Sio/homelab">GitHub repo</a></td>
        </tr>
         <tr>
            <td><img class="customEntitityAlbum" style="background-color:transparent" src="hugo.png"/></td>
            <td>
              My website
              {{< badge >}}
                Active
              {{< /badge >}}
            </td>
            <td>My simple static website powered by <a target="_blank" href="https://gohugo.io/">Hugo</a> & <a target="_blank" href="https://blowfish.page/">Blowfish</a>.</td>
            <td><a target="_blank" href="https://github.com/justin-de-sio/my-blog">Github repo</a></td>
        </tr>
        <tr>
            <td><img class="customEntitityAlbum" style="background-color:transparent" src="cicd.png"/></td>
            <td>
              JSTAcademy
              {{< badge >}}
                Achived
              {{< /badge >}}
            </td>
            <td>
                <p>JST Academy is an interactive e-learning platform providing structured courses with lessons and quizzes. It uses a modern tech stack with <strong>Angular</strong> for the frontend and <strong>Java Spring Boot</strong> for the backend.</p>
                <p>Its infrastructure relies on two automated <strong>CI/CD pipelines</strong> managed through GitLab:</p>
                <ul>
                <li>The first pipeline builds Docker images, runs integration tests, and deploys to <strong>AWS EC2</strong> using Ansible.</li>
                <li>The second pipeline provisions and manages cloud infrastructure on <strong>AWS EC2</strong> using <strong>Terraform</strong>.</li>
                </ul>
            </td>
                <td><a target="_blank" href="https://gitlab.com/jstacademy">GitLab repo</a></td>
        </tr>

</tbody>

</table>
