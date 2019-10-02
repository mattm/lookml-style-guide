# Mazur's LookML Style Guide

Howdy! I'm [Matt Mazur](https://mattmazur.com/) and I'm a data analyst who has worked at several companies to help them use data to grow their businesses. I've been working in [Looker](http://looker.com/) almost daily since June 2017 and over time have developed various preferences for how to write clean, maintainable LookML. This guide is an attempt to document those preferences.

My goal with this guide is twofold: first, to share what I've learned so that it may help other LookML developers. Second, I'd love to get feedback from you all about how to improve how I write LookML. I'll incorporate any feedback from you all in here and give you credit where appropriate. You can find me on Twitter at [@mhmazur](https://twitter.com/mhmazur) (where I'll also be tweeting about updates to this guide) or by email at matthew.h.mazur@gmail.com.

If you're interested in this topic, you may also enjoy my [SQL Style Guide](https://github.com/mattm/sql-style-guide), my [Matt On Analytics](http://eepurl.com/dITJS9) newsletter, and [my blog](https://mattmazur.com/category/analytics/) where I write about analytics and data analysis.

## Guidelines

### Omit unnecessary parameters

When possible, I try to take advantage of Looker's default parameter values to minimize how much LookML I have to write. I think the LookML looks cleaner that way, even if it sometimes comes at the cost of being less explicit. A few examples:

Dimensions will default to referencing a column that matches the name of the dimension, so you can leave the `sql` parameter out in lot of cases:

```lookml
-- Good
dimension: is_first_billing {
  description: "..."
  type: yesno
}

-- Bad
dimension: is_first_billing {
  description: "..."
  type: yesno
  sql: ${TABLE}.is_first_billing ;;
}
```

Similarly, no need to include a `label` most of the time because Looker will automatically case titleize it:

```lookml
-- Good
dimension: is_first_billing {
  description: "..."
  type: yesno
}

-- Bad
dimension: is_first_billing {
  label: "Is First Billing"
  description: "..."
  type: yesno
}
```

Another example is excluding `type: left_outer` from joins because left joining is the default behavior:

```lookml
-- Good
explore: companies {
  join: billings {
    relationship: one_to_many
    sql_on: ${companies.company_id} = ${billings.company_id} ;;
  }
}

-- Bad
explore: companies {
  join: billings {
    relationship: one_to_many
    type: left_outer
    sql_on: ${companies.company_id} = ${billings.company_id} ;;
  }
}
```
