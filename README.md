# researchthroughdesign.org

A static export of researchthroughdesign.org's various wordpress instances.

**docker-compose.yml**

```yaml
volumes:
  mysql:

services:
  mysql:
    image: mysql:5.7
    platform: linux/amd64
    restart: unless-stopped
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_USER: user
      MYSQL_PASSWORD: secret
    volumes:
      - mysql:/var/lib/mysql
  
  app:
    # image: php:5.6-apache
    image: wordpress:php5.6-apache
    platform: linux/amd64
    restart: unless-stopped
    ports:
      - 8080:80
    volumes:
      - ./public_html:/var/www/html/
```

- The various databases manually imported into that mysql container.
  - You might need to `SET sql_mode = '';` because of WordPress' date format [link](https://tableplus.com/blog/2019/10/incorrect-date-value-0000-00-00-date-datetime.html)
- The wp-config.php files are updated to use the `mysql` container and its credentials
- The databases themselves are updated (`wp_options`) to change the site and home URLs to reflect a `localhost:8080` deployment
- The proxy (below) is used to access and rewrite any URLs on the page which reference itself absolutely
- This allows the `wget` command to create a relative-url-based archive, that is now hosted on GitHub pages

**proxy.js**

```js
#!/usr/bin/env deno run -A
const contentTypes = [
  'text/html'
]

const rewrites = [
  'https://rtd.openlab.dev',
  'http://www.researchthroughdesign.org',
  'https://www.researchthroughdesign.org',
  'http://researchthroughdesign.org',
  'https://researchthroughdesign.org',
]

function process(text) {
  let output = text
  for (const url of rewrites) {
    output = output.replaceAll(url, 'https://rtd.openlab.dev')
  }
  return output
}

Deno.serve({ port:9000 }, async request => {
  const url = new URL('.' + new URL(request.url).pathname, 'https://rtd.openlab.dev')
  const res = await fetch(url)
  const type = res.headers.get('content-type')
  if (contentTypes.every(t => !type?.includes(t))) return res
  return new Response(process(await res.text()), {
    headers: res.headers,
    status: res.status
  })
})
```

**run.sh**

```bash
#!/usr/bin/env sh

wget \
  --mirror \
  --convert-links \
  --adjust-extension \
  -o wget.log \
  https://rtd.openlab.dev
```


## notes

- several pages in 2015 point to www.johnvines.eu which no longer exists