+++ 
archetype = "moc" 
title = "Packages" 
menuPre = '<i class="fa-solid fa-box-open"></i> '
weight = 9
alwaysopen = false
featured_image = "images/packages.png"
[_build]
  render = "always"
  list = "always"
  publishResources = true
+++

{{% children type="card" description="true" %}}