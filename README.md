# Mazur's LookML Style Guide

Howdy! I'm [Matt Mazur](https://mattmazur.com/) and I'm a data analyst who has worked at several companies to help them use data to grow their businesses. I've been working in [Looker](http://looker.com/) almost daily since June 2017 and over time have developed various preferences for how to write clean, maintainable LookML. This guide is an attempt to document those preferences.

My goal with this guide is twofold: first, to share what I've learned so that it may help other LookML developers. Second, I'd love to get feedback from you all about how to improve how I write LookML. I'll incorporate any feedback from you all in here and give you credit where appropriate. You can find me on Twitter at [@mhmazur](https://twitter.com/mhmazur) (where I'll also be tweeting about updates to this guide) or by email at matthew.h.mazur@gmail.com.

If you're interested in this topic, you may also enjoy my [SQL Style Guide](https://github.com/mattm/sql-style-guide), my [Matt On Analytics](http://eepurl.com/dITJS9) newsletter, and [my blog](https://mattmazur.com/category/analytics/) where I write about analytics and data analysis.

## Guidelines

### All visible dimensions should have a description

It's tempting to omit descriptions for dimensions whose purpose seems obvious from the name, but what may be obvious to you may not be obvious to other analysts. You can almost always add clarity by adding a meaningful description to a dimension.

```lookml
# Good
dimension: promotion_description {
  description: "A description of any promotion that was applied to this billing. For example, "Plus Plan 25%" or "Non-profit Discount". A billing can only have 1 promotion max."
}

# Bad: A description that restates the dimension name
dimension: promotion_description {
  description: "A description of the promotion"
}

# Bad: No description
dimension: promotion_description {}
```

The only exception is dimensions that won't be visible to the user when performing analyses, for example primary or foreign keys that do not need to be exposed to users. For these type of dimensions, descriptions are optional.

### Omit unnecessary parameters

When possible, I try to take advantage of Looker's default parameter values to minimize how much LookML I have to write. I think the LookML looks cleaner that way, even if it sometimes comes at the cost of being less explicit. A few examples:

Dimensions will default to referencing a column that matches the name of the dimension, so you can leave the `sql` parameter out in lot of cases:

```lookml
# Good
dimension: is_first_billing {
  type: yesno
}

# Bad
dimension: is_first_billing {
  type: yesno
  sql: ${TABLE}.is_first_billing ;;
}
```

Similarly, no need to include a `label` most of the time because Looker will automatically case titleize it:

```lookml
# Good
dimension: is_first_billing {
  type: yesno
}

# Bad
dimension: is_first_billing {
  label: "Is First Billing"
  type: yesno
}
```

Another example is excluding `type: left_outer` from joins because left joining is the default behavior:

```lookml
# Good
explore: companies {
  join: billings {
    relationship: one_to_many
    sql_on: ${companies.company_id} = ${billings.company_id} ;;
  }
}

# Bad
explore: companies {
  join: billings {
    relationship: one_to_many
    type: left_outer
    sql_on: ${companies.company_id} = ${billings.company_id} ;;
  }
}
```

### Naming count measures

Use a [singular noun](https://twitter.com/jgkite/status/1171845537311707136) representing whatever thing you're measuring suffixed with `_count`. For example, `company_count`, `user_count`, `beacon_count`, `purchase_count`, etc. Avoid the default `count` measure name that's created when you use Looker's "Create View From Table" feature. This helps avoid ambiguity that might arise when people see just "Count" in analyses and visualizations.

```lookml
# Good
view: companies {
  measure: company_count {
    type: count
  }
}

# Bad
view: companies {
  measure: count {
    type: count
  }
}
```

### Naming sum measures

For sums, prefix the measure name with `total_`. For example, `total_payments`, `total_taxes`, etc:

```lookml
# Good
view: payments {
  measure: total_payments {
    type: sum
    sql: ${TABLE}.amount ;;
  }
}
```

One exception is when you're summing a column that already represents a count. For example, if you have a `companies` table and there's a `users` column that representing the number of users a company has, you should name the measure like you would a count because the measure represents the number of users, even though it's using a sum behind the scenes.

```lookml
# Good
view: companies {
  measure: user_count {
    type: sum
    sql: ${TABLE}.users ;;
  }
}

# Bad
view: companies {
  measure: total_users {
    type: sum
    sql: ${TABLE}.users ;;
  }
}
```

## Credits

A lot of these preferences were developed in conjection with others during my time at Help Scout and Automattic. Huge thanks in particular to to Eli Overbey, Simon Ouderkirk, and Jen Wilson for the many discussions that have influenced my thinking on writing LookML.
