## How to set it up!
1. Clone this repo into your Organization.

2. Once done, Please allow CodeQL to complete its initial scan. This should take around 2-3 minutes.
<img width="1158" alt="image" src="https://github.com/jackgkafaty/GitHub_TestRepo/assets/50452463/8bb466b5-d765-44e7-88e9-b1f8ccfc04fb">

3. Once completed, create a new branch! Let's call this branch **Development**

4. Once done, within that branch please access the **showProductReviews.ts** file located under GitHub_TestRepo/blob/Development/routes/showProductReviews.ts

5. Edit the code, replace it with the code below, and commit the changes to the Development branch.

```typescript
/*
 * Copyright (c) 2014-2023 Bjoern Kimminich & the OWASP Juice Shop contributors.
 * SPDX-License-Identifier: MIT
 */

import utils = require('../lib/utils')
import challengeUtils = require('../lib/challengeUtils')
import { Request, Response, NextFunction } from 'express'
import { Review } from 'data/types'

const challenges = require('../data/datacache').challenges
const security = require('../lib/insecurity')
const db = require('../data/mongodb')

// Blocking sleep function as in native MongoDB
// @ts-expect-error test
global.sleep = (time: number) => {
  // Ensure that users don't accidentally dos their servers for too long
  if (time > 2000) {
    time = 2000
  }
  const stop = new Date().getTime()
  while (new Date().getTime() < stop + time) {
    ;
  }
}

module.exports = function productReviews () {
  return (req: Request, res: Response) => {
    const id = utils.disableOnContainerEnv() ? Number(req.params.id) : req.params.id

    // Measure how long the query takes, to check if there was a NoSQL dos attack
    const t0 = new Date().getTime()
    db.reviews.find({ $where: 'this.product == ' + id }).then((reviews: Review[]) => {
      const t1 = new Date().getTime()
      challengeUtils.solveIf(challenges.noSqlCommandChallenge, () => { return (t1 - t0) > 2000 })
      const user = security.authenticatedUsers.from(req)
      for (let i = 0; i < reviews.length; i++) {
        if (user === undefined || reviews[i].likedBy.includes(user.data.email)) {


        }
      }
      res.json(utils.queryResultToJson(reviews))
    }, () => {
      res.status(400).json({ error: 'Wrong Params' })
    })
  }
}
```

<img width="955" alt="image" src="https://github.com/jackgkafaty/GitHub_TestRepo/assets/50452463/8029a446-fed4-4d83-8380-fd387bdc7c31">


