---
layout: page
title: Projects
permalink: /projects/
---


  <ul class="posts">
        {% for post in site.posts %}
        <li>
            <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
            <span class="posted-date">{{ post.date | date: "%b %-d, %Y" }}</span>
        </li>
        {% endfor %}
    </ul>




[ames]: www.nasa.gov/centers/ames/
[twitter]: https://www.twitter.com/teme
[linkedin]: www.linkedin.com/in/temesghen/
[bitbucket]: https://bitbucket.org/lememta
[rse]: www.ti.arc.nasa.gov/tech/rse/
[mine]: www.ti.arc.nasa.gov/profile/tkahsaia/
[cmu]: www.cmu.edu/silicon-valley/
