---
title: Site Setup
issue_id: 1
---

The time is (almost) here for painless static site creation and hosting, so I
gave it a try. I'm documenting the steps I took to get this site running for
future reference, or in case someone wants to take the same path.

### Requirements

I didn't want to compromise on these points:

1. The site's canonical URL is [https://www.viktorroytman.com/][0]
  - That means it must be served over [HTTPS][1].
  - The `www` [is not optional][20].
2. It's a [static site][2], with maintenance-free hosting, more or less.
3. The site's source code is available on [GitHub][3] ([and it is!][4]).

With these conditions, my first thought was to use [GitHub Pages][5], but
unfortunately that doesn't allow the use of a custom SSL certificate (as
discussed (quite fruitlessly) [here][6] (and yes, I know you can fake it with
[Cloudflare][7])).

So I settled on [GitLab Pages][8]... mirroring a repository on GitHub. It makes
sense, I promise.

### Buying the domain

I bought my domain at [Namecheap][9]. These days you can buy your domain using
a number of services, and they're all probably effectively the same (though
it's in bad taste to use [GoDaddy][10]).

### Creating a repository on GitLab (and GitHub)

The GitLab Pages documentation walks you through creating a repository for your
site. The only extra step for me was to make it a [mirror][11] of the GitHub
repository. I want the site's main repository on GitHub so that I can make use
of GitHub Issues for comments. I'll talk about this in another post...

#### Aside: Why GitHub?

I know, I know. The [GitHub Monoculture][12]. I do think it's a problem, but it
does have one benefit &mdash; newcomers have a generally-accepted "standard"
when it comes to software development online, particularly with regard to [free
and open-source software][13] and [Git][14]. I think of it a bit like
[Ubuntu][15]; while I can't stomach Ubuntu myself, I can't deny the value in
having a widely-used examplar as a "beginner" Linux distribution.

### Setting the DNS records

Simple enough:

- An A record with host `@` and value `104.208.235.32` (from
  [GitLab's guide][16]).
- A CNAME record with host `www` and value `viktor-roytman.gitlab.io.`

### Adding an SSL certificate

GitLab has a long-winded (and somewhat out-of-date) guide [here][17] for
serving your site via HTTPS. Hopefully GitLab will introduce better integration
with [Let's Encrypt][18] in the future... Here is my version of the
instructions:

1. On the "Pages" section of the GitLab repository settings, add the URL of
your site. Assuming you have the right DNS records, this will serve your site
via HTTP.
2. Install and run Let's Encrypt's [Certbot][19] tool with the following
command. Note that I had to specify both the `www` and non-`www` versions of my
URL.     
                                                                             
   ```                                                                       
   $ sudo certbot certonly --manual -d www.viktorroytman.com -d viktorroytman.com
   ```
3. After agreeing to a few things and supplying a contact e-mail address, you
will see a message like the following. It will tell you to host some content at
a page on your domain in order to prove that you control that domain. If you
take too long, you'll get an error message about an expired [nonce][21], but
you can always try again by re-running the command.

   ```
    Make sure your web server displays the following content at
    http://YOURDOMAIN.org/.well-known/acme-challenge/<long_string>
    before continuing:

    <another_long_string>
   ```
4. Create a file in your repository (I called it `acme-challenge`) that
contains the content that Let's Encrypt wants to see. The `layout: null` and
`permalink` stuff is specific to [Jekyll][22], the static site generator I
used, but the same principles should apply to most static site generators.

   ```
    ---
    layout: null
    permalink: /.well-known/acme-challenge/<long_string>
    ---

    <another_long_string>
   ```
5. Once you've confirmed that you are actually hosting this new file, go back
to the terminal window where you ran the `certbot` command and hit the `Enter`
key to have Let's Encrypt look for your file. If all goes well, you should have
an SSL certificate valid for 90 days now. However, if you (like me) want to use
both `www` and non-`www` URLs, you will have to host a second file much like
the first one. You can see for yourself at my repository: [here][23] and
[here][24].

6. Go back to the "Pages" section of the GitLab repository settings and delete
the entry you made before (without an SSL certificate). Recreate the entry, but
now fill in the "Certificate (PEM)" and "Key (PEM)" fields using the files
generated by Certbot. I had to make two entries, one for `www` and one for
non-`www`. For me, those files were at these locations:

   ```
    /etc/letsencrypt/live/www.viktorroytman.com/fullchain.pem
    /etc/letsencrypt/live/www.viktorroytman.com/privkey.pem
   ```

The good: You have hosted your free static site with custom domain via HTTPS!

The bad: You will need to [renew][25] your certificate at least once every 90 days.

### Redirecting "bad" versions of the URL to the good one

I've gone to all this trouble to get `www` and `https://` working, and I intend
to have people visit the correct URL. Ideally, I would be able set redirect
rules on the web server, but I can't do that with GitLab Pages. Instead, I have
to make do with GitLab's [suggestions][26]. You can see how I did it [here][27].

### Was it worth it?

I've written quite a few words here... What have I to show for it? Personal
satisfaction, I suppose. Plus, `http://viktorroytman.com` just doesn't look
right, does it?

[0]: https://www.viktorroytman.com/
[1]: https://en.wikipedia.org/wiki/HTTPS
[2]: https://en.wikipedia.org/wiki/Static_web_page
[3]: https://github.com/
[4]: https://github.com/vr2262/viktor-roytman.gitlab.io
[5]: https://pages.github.com/
[6]: https://github.com/isaacs/github/issues/156
[7]: https://www.cloudflare.com/
[8]: https://pages.gitlab.io/
[9]: https://www.namecheap.com/
[10]: https://en.wikipedia.org/wiki/GoDaddy#Controversies
[11]: https://docs.gitlab.com/ee/workflow/repository_mirroring.html#pulling-from-a-remote-repository
[12]: http://nedbatchelder.com/blog/201405/github_monoculture.html
[13]: https://en.wikipedia.org/wiki/Free_and_open-source_software
[14]: https://git-scm.com/
[15]: https://en.wikipedia.org/wiki/Ubuntu_(operating_system)
[16]: https://about.gitlab.com/2016/04/07/gitlab-pages-setup/#custom-domains
[17]: https://about.gitlab.com/2016/04/11/tutorial-securing-your-gitlab-pages-with-tls-and-letsencrypt/
[18]: https://letsencrypt.org/
[19]: https://certbot.eff.org/
[20]: http://www.yes-www.org/
[21]: https://en.wikipedia.org/wiki/Cryptographic_nonce
[22]: https://jekyllrb.com/
[23]: https://github.com/vr2262/viktor-roytman.gitlab.io/blob/master/acme-challenge
[24]: https://github.com/vr2262/viktor-roytman.gitlab.io/blob/master/acme-challenge-www
[25]: https://certbot.eff.org/docs/using.html#renewing-certificates
[26]: https://about.gitlab.com/2016/04/11/tutorial-securing-your-gitlab-pages-with-tls-and-letsencrypt/#redirecting
[27]: https://github.com/vr2262/viktor-roytman.gitlab.io/blob/3aa149a9f44184dd28268662dee8a5371e368189/_layouts/default.html#L11-L20