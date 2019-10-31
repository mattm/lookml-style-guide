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

### Remove unused fields and views

If you've taken advantage of Looker's "Create View from Table" feature, you may have wound up with a lot of unused views and dimensions. It's tempting to leave them in place because you may need them one day, but my preference is to remove them entirely. The cost of creating new views and dimensions is small compared to the cognitive overhead resulting from lots of unused views and hidden fields that aren't used in any analyses.

### Omit unnecessary parameters

When possible, I try to take advantage of Looker's default parameter values to minimize how much LookML I have to write. I think the LookML looks cleaner that way, even if it sometimes comes at the cost of being less explicit. A few examples:

Dimensions will default to referencing a column that matches the name of the dimension, so you can leave the `sql` parameter out in lot of cases:

```lookml
# Good
dimension: is_first_billing {
  description: "..."
  type: yesno
}

# Bad
dimension: is_first_billing {
  description: "..."
  type: yesno
  sql: ${TABLE}.is_first_billing ;;
}
```

Similarly, no need to include a `label` most of the time because Looker will automatically case titleize it:

```lookml
# Good
dimension: is_first_billing {
  description: "..."
  type: yesno
}

# Bad
dimension: is_first_billing {
  label: "Is First Billing"
  description: "..."
  type: yesno
}
```

And no need to include `type: string` which is the default type:

```lookml
# Good
dimension: name {
  description: "..."
}

# Bad
dimension: name {
  description: "..."
  type: string
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

### Yes/No dimensions should be exposed as case/when string dimensions

Rather than use a dimension like `is_paying` in an analysis, it's better to hide it and make a derived `paying_status` string dimension available. This makes it abundently clear what the yes and no values represent especially when displayed in visualizations (pivoting a measure on a yes/no and seeing Yes and No in a legend is often unclear).

```lookml
# Good: Hiding the yes/no dimension and creating a LookML case/when dimension based on it
dimension: is_paying {
  type: yesno
  hidden: yes
}

dimension: paying_status {
  description: "Whether the company is currently paying or non-paying"
  case:
    when: {
      sql: ${is_paying} ;;
      label: "Paying"
    }
    else: "Non-Paying"
  }
}

# Bad: Using SQL to determine the string values

# You want to use LookML case/when so that Looker displays a dropdown that users can select from when filtering
# Otherwise Looker will simply display a text field which is less easy to use.

dimension: is_paying {
  type: yesno
  hidden: yes
}

dimension: paying_status {
  description: "Whether the company is currently paying or non-paying"
  sql: if(${is_paying}, "Paying", "Non-Paying") ;;
}

# Bad: Exposing the yes/no dimension to end-users
dimension: is_paying {
  description: "Whether the company is currently a paying customer"
  type: yesno
}
```

### Alphabetize dimensions, then alphabetize measures

Looker doesn't care about the order of the fields within a view, but it makes it easier to find specific fields if you simply list dimensions alphabetically then list measures alphabetically:

```lookml
# Good
view: companies {
  dimension: id {
    description: "..."
    type: number
  }
  
  dimension: has_closed {
    description: "..."
    type: yesno
  }

  dimension: name {
    description: "..."
  }

  measure: closed_count {
    description: "..."
    type: count
    filters: {
      field: has_closed
      value: "yes"
    }
  }

  measure: company_count {
    description: "..."
    type: count
  }
}

# Bad
view: companies {
  measure: closed_count {
    description: "..."
    type: count
    filters: {
      field: has_closed
      value: "yes"
    }
  }

  dimension: id {
    description: "..."
    type: number
  }

  dimension: name {
    description: "..."
  }

  measure: company_count {
    description: "..."
    type: count
  }

  dimension: has_closed {
    description: "..."
    type: yesno
  }
}
```

### Every view should have a primary key defined

It's [required](https://docs.looker.com/data-modeling/learning-lookml/working-with-joins#primary_keys_required) for [symmetric aggregates](https://discourse.looker.com/t/symmetric-aggregates/261) to work correctly.

```lookml
# Good
view: companies {
  dimension: company_id {
    description: "..."
    primary_key: yes
  }
}
```

If the table you're working with does not have a primary key, you can always create a primary key dimension by concatenating other columns together as outlined in [the docs](https://docs.looker.com/reference/field-params/primary_key). Ideally though you would transform the table prior to analyzing it in Looker using a tool like [dbt](http://getdbt.com/) and create a primary key there if needed, eliminating the need to concatenate fields in Looker (more on dbt in a future tip).

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

A lot of these preferences were developed in conjection with others during my time at Help Scout and Automattic. Huge thanks in particular to to Eli Overbey, Simon Ouderkirk, Anna Elek, and Jen Wilson for the many discussions that have influenced my thinking on writing LookML.
