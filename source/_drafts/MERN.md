MERN 踩坑记录

学完 fso 来做个全栈 app

# apollo

apollo proxy relative url

https://gist.github.com/wickedev/8dfbc026e2142a5e82810a2d7a7156a9

# Playwright

On Windows, use

```bash
npx playwright test ./tests/whateverthetestdatasnameis.spec.js
```

instead of

```bash
npx playwright test .\tests\whateverthetestdatasnameis.spec.js
```

# ts node test tsx directory

```ts
import * as fs from 'fs'
import * as path from 'path'

const testDir = path.join(__dirname, '')

fs.readdirSync(testDir).forEach((file) => {
  if (file.endsWith('.test.ts')) {
    require(path.join(testDir, file))
  }
})
```
