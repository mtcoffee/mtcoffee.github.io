---
title: Cleaning up Stale CI Relationships in ServiceNow
---

# Summary 
Cleaning up stale CI relationships in ServiceNow is a common challenge that is unfortunately not well addressed by ServiceNow out of the box. To quote a colleague of mine, ServiceNow's guidance is basically "I dunno, you guys figure it out"😆. More officially they do state that this is something you [need to DIY](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB1112334).  Doug Schulze of ServiceNow does offer an [unofficial solution](https://www.servicenow.com/community/itom-articles/ci-lifecycle-part-deux-the-sequel/ta-p/2321224) which offers good guidance.

I decided to try coming up with a simple scheduled script that could serve has as a starting point. Ultimately something like this would be best in a script include.

# Solution
Some folks have gone with a simple, [delete all relationships where the parent or child is retired](https://www.servicenow.com/community/service-management-forum/deleted-ci-relationship-if-ci-is-retired-or-decommissioned/m-p/395414), but you might risk losing important connections like a reference to a process document. It may be best to limit the cleanup to the relationships  needed to show clean Service Maps.

This script will:
* Find/Delete all relationships for retired items where parent is hardware 
* Find/Delete all relationships where parent is cmdb_ci_appl based class and child is retired hardware
* Find/Delete all relationships where parent is cmdb_ci_service based class and child ci is retired

<script src="https://gist.github.com/mtcoffee/4b389fb3ce69801f223e178a13cf6916.js"></script>
