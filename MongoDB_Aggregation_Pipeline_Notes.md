## Mongo Aggregation Pipeline Questions

01. Find how many user are active? <br>

```javascript 
[
  {
    // matches the query from DB
    $match: {
      isActive: false
    }
  },
  {
    // returns the count records of prev stage
    $count: "activeUsers"
  }
]
```

02. What is the average age of all users? <br>
```javascript
[
  {
    // first grp all the records
    // use identifier as "null" to select all records
    $group: {
      _id: null,
      // pass in a field to hold avg Age
      avgAge: {
        // accumulator and expression to evaluate upon
        $avg: "$age",
      },
    },
  },
]
```

03. List top 5 common favourite fruits among the users? <br>
```javascript
[
  {
    // grp based on favFruit
    $group: {
      _id: "$favoriteFruit",
      // get count of each grp
      count: {
	// $sum is an accumulatort
	// accumulator do not calc fresh values
	// they just build up upon previous values

        // this increases the cnt by on
        // any time it sees a new record
        $sum: 1,
      },
    },
  },
  {
    // sort the count field
    // count field is not in the record
    // but can be used becoz it is present in above stage
    $sort: {
      // 1 for Ascen, -1 for Descen
      count: -1,
    },
  },
  // limit the no. of records required
  {
    $limit: 2,
  },
]
```

04. Find the total number of males and females <br>
```javascript
[
  {
    $group: {
      _id: "$gender",
      total: {
        $sum: 1,
      },
    },
  },
]
```

05. Which country has the highest number of registered users? <br>
```javascript
[
  {
    $group: {
      _id: "$company.location.country",
      Registered_Users: {
        $sum: 1,
      },
    },
  },
  {
    $sort: {
      Registered_Users: -1,
    },
  },
  {
    $limit: 5,
  },
]
```

06. List all the unique eye colors present in the collection <br>
```javascript
[
  {
    $group: {
      _id: "$eyeColor",
    },
  },
]
```

07. What is the avg number of tags per user? <br>
```javascript
// 1st way
[
  {
    // the main challenge was access the values in tags[]
    // use unwind op. to unwind the array
    $unwind: "$tags",
  },
  {
    $group: {
      _id: "$name",
      noOfTags: {
        $sum: 1,
      },
    },
  },
  {
    $group: {
      _id: null,
      avgCount: {
        $avg: "$noOfTags",
      },
    },
  },
]
```
```javascript
// 2nd way
[
  {
    // addFields op add new fields in document
    $addFields: {
      noOfTags: {
        // $size op gives length of array
        $size: {
          // $ifNull allows to provide a fallBack if
          // "$tags is not present"
          $ifNull: ["$tags", []],
        },
      },
    },
  },
  {
    $group: {
      _id: null,
      avg: {
        $avg: "$noOfTags",
      },
    },
  },
]
```

08. How many users hava "enim" as their one of the tags <br>
```javascript
[
  {
    $match: {
      tags: "id",
    },
  },
  {
    $count: "users",
  },
]
```

09. What is the name and age of users who are inactive and have "velit" tag <br>
```javascript
[
  {
    // match allows to match multiple attributes at once
    $match: {
      isActive: false,
      tags: "velit",
    },
  },
  {
    // we can use project to select what all fields should be returned
    $project: {
      name: 1,
      age: 1,
    },
  },
]
```

10. find users with +1 (947) as starting of their phone number <br>
```javascript
[
  {
    $match: {
      "company.phone": /^\+1 \(947\)/,
    },
  },
  {
    $count: "usersWith+1(947)",
  },
]
```

11. Find the top three most recently registerd users and return their name and active status <br>
```javascript
[
  {
    $sort: {
      registered: -1,
    },
  },
  {
    $limit: 3,
  },
  {
    $project: {
      name: 1,
      isActive: 1,
      registered: 1,
    },
  },
]
```

12. Categorize users by their fav. fruit <br>
```javascript
[
  {
    $group: {
      _id: "$favoriteFruit",
      numOfUsers: {
        // push is an accumulator that pushes fields in an array
        $push: "$name",
      },
    },
  },
]
```

13. How many users hava "ad" as their second tag in list of tags? <br>
```javascript
[
  {
    $match: {
      // we can use arr.index to find a specific value inside an array
      // so this gives all the docs that have "ad" at first index in the tags array
      "tags.1": "ad",
    },
  },
  {
    $count: "noOfUsers",
  },
]
```

14. Find users that have both "enim" and "ad" as their tags <br>
```javascript
[
  {
    $match: {
      tags: {
        // all selects a doc only when all the specified fields in the arr match
        $all: ["id", "enim"],
      },
    },
  },
]
```

15. List companies located in USA with their resp. user count <br>
```javascript
[
  {
    $match: {
      "company.location.country": "USA",
    },
  },
  {
    $group: {
      _id: null,
      userCount: {
        $sum: 1,
      },
    },
  },
]
```

16. left join author_id and _id <br>
```javascript
// $lookup is equivalent of left join
[
  {
    $lookup: {
      from: "author",
      localField: "author_id",
      foreignField: "_id",
      as: "author_details",
    },
  },
  // lookup returns an array of object
  {
    $addFields: {
      author_details: {
        // we can use $first or $arrayElemAt to extract the object
        // and add it as specific field in doc
        // $first: "$author_details",
        $arrayElemAt: ["$author_details", 0],
      },
    },
  },
]
```

<!-- /* 			END			 */ -```
