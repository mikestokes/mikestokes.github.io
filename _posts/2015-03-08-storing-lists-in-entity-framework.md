---
layout: post
title: Entity Framework and Lists
---

By default [Entity Framework](http://www.asp.net/entity-framework) (as of version 6.x) does not support serialization of Lists to a database field. This is a fraustrating limitation as this can be a useful mechanism to store 

For example, the following is *not* possible by default in Entity Framework:

```c#
[Table("YourTableName")]
public class YourEntity
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

	public List<string> YourList { get; set; }
}

```

## Serializing using Complex Types

After searching I cam upon a solution from [Bernhard Kircker](http://stackoverflow.com/questions/11985267/entity-framework-options-to-map-list-of-strings-or-list-of-int-liststring) that we have put into production and has been working very well for us. I'll outline here what we did, with thanks to Bernhard.

### Creating a Custom ICollection
