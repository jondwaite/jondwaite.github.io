---
title: And we’re back…
author: Jon Waite
type: post
date: 2022-04-16T10:02:07+00:00
url: /2022/04/and-were-back/
categories:
  - Uncategorized

---
So I had an &#8216;interesting&#8217; email from Oracle cloud (OCI) which I almost missed:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="761" height="800" src="https://kiwicloud.ninja/wp-content/uploads/2022/04/image-761x800.png" alt="" class="wp-image-68982" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/04/image-761x800.png 761w, https://kiwicloud.ninja/wp-content/uploads/2022/04/image-285x300.png 285w, https://kiwicloud.ninja/wp-content/uploads/2022/04/image-768x807.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/04/image-143x150.png 143w, https://kiwicloud.ninja/wp-content/uploads/2022/04/image.png 1001w" sizes="(max-width: 761px) 100vw, 761px" /> </figure> 

As this site (kiwicloud.ninja) is currently hosted on the &#8216;Free Tier&#8217; of Oracle Cloud I was a bit concerned, but figured the odds of being impacted were low and I &#8216;should be alright&#8217;.

Unfortunately &#8211; turned out it wasn&#8217;t alright, OCI had &#8216;unintentionally reclaimed&#8217; the public IP address which the site used and I had to jump into the Oracle dashboard and navigate through to assign a new reserved public IP address to the site.

Then I had to find my DNS registration logins and update those so that the domain pointed correctly to the updated public IP (unfortunately my previous public IP had already been reallocated and so was unavailable to reassign).

So everything is fixed and the site is now back on the air, but it does have me re-evaluating my choice of where the best place is for this site to live, and how I should monitor it in future to get better notifications when things go wrong.

Jon