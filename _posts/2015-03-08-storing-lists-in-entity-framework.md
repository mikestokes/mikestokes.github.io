---
layout: post
title: Entity Framework and Lists
category: programming
tags: [c#, entity framework]
---

By default [Entity Framework](http://www.asp.net/entity-framework) (as of version 6.x) does not support serialization of Lists to a database field.

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

## Serializing Lists using Entity Framework Complex Types

After searching I came upon a rather simple (and re-usable) solution from [Bernhard Kircker](http://stackoverflow.com/questions/11985267/entity-framework-options-to-map-list-of-strings-or-list-of-int-liststring) that we have put into production and has been working very well for us. I'll outline here what we did, with thanks to Bernhard for the solution.

With Bernhard's solution, the above Entity class can simply be changed and Entity Framework will take care of the rest for you:

```c#
[Table("YourTableName")]
public class YourEntity
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

	[Column("YourColumnName", TypeName ="ntext")]
	public virtual PersistableStringCollection YourList { get; set; }
}
```

### Using the Collection

You can use the collection very simply - just like you would a normal .NET Framework enumerable, for example:

Adding a value: `YourEntity.YourList.Add(value);`

Clearing the collection: `YourEntity.YourList.Clear();`

### Creating a Custom ICollection

So, what is needed to achieve this?

We need a simple base ICollection class which can be extended for each type we wish to serialize (string, int etc). Luckily Bernhard has done most of the heavy lifting for us here and I've included this below for completeness:

```c#
/// <summary>
/// Baseclass that allows persisting of scalar values as a collection (which is not supported by EF 4.3)
/// </summary>
/// <typeparam name="T">Type of the single collection entry that should be persisted.</typeparam>
[ComplexType]
public abstract class PersistableScalarCollection<T> : ICollection<T>
{
	// use a character that will not occur in the collection.
	// this can be overriden using the given abstract methods (e.g. for list of strings).
	const string DefaultValueSeperator = "|";

	readonly string[] DefaultValueSeperators = new string[] { DefaultValueSeperator };

	/// <summary>
	/// The internal data container for the list data.
	/// </summary>
	private List<T> Data { get; set; }

	public PersistableScalarCollection()
	{
		Data = new List<T>();
	}

	/// <summary>
	/// Implementors have to convert the given value raw value to the correct runtime-type.
	/// </summary>
	/// <param name="rawValue">the already separated raw value from the database</param>
	/// <returns></returns>
	protected abstract T ConvertSingleValueToRuntime(string rawValue);

	/// <summary>
	/// Implementors should convert the given runtime value to a persistable form.
	/// </summary>
	/// <param name="value"></param>
	/// <returns></returns>
	protected abstract string ConvertSingleValueToPersistable(T value);

	/// <summary>
	/// Deriving classes can override the string that is used to separate single values
	/// </summary>
	protected virtual string ValueSeperator
	{
		get
		{
			return DefaultValueSeperator;
		}
	}

	/// <summary>
	/// Deriving classes can override the string that is used to separate single values
	/// </summary>
	protected virtual string[] ValueSeperators
	{
		get
		{
			return DefaultValueSeperators;
		}
	}

	/// <summary>
	/// DO NOT Modify manually! This is only used to store/load the data.
	/// </summary>
	public string SerializedValue
	{
		get
		{
			var serializedValue = string.Join(ValueSeperator.ToString(),
				Data.Select(x => ConvertSingleValueToPersistable(x))
				.ToArray());
			return serializedValue;
		}
		set
		{
			Data.Clear();

			if (string.IsNullOrEmpty(value))
			{
				return;
			}

			Data = new List<T>(value.Split(ValueSeperators, StringSplitOptions.None)
				.Select(x => ConvertSingleValueToRuntime(x)));
		}
	}

	#region ICollection<T> Members

	public void Add(T item)
	{
		Data.Add(item);
	}

	public void Clear()
	{
		Data.Clear();
	}

	public bool Contains(T item)
	{
		return Data.Contains(item);
	}

	public void CopyTo(T[] array, int arrayIndex)
	{
		Data.CopyTo(array, arrayIndex);
	}

	public int Count
	{
		get { return Data.Count; }
	}

	public bool IsReadOnly
	{
		get { return false; }
	}

	public bool Remove(T item)
	{
		return Data.Remove(item);
	}

	#endregion

	#region IEnumerable<T> Members

	public IEnumerator<T> GetEnumerator()
	{
		return Data.GetEnumerator();
	}

	#endregion

	#region IEnumerable Members

	IEnumerator IEnumerable.GetEnumerator()
	{
		return Data.GetEnumerator();
	}

	#endregion
}
```

### List of Strings

For example, if we need to serialize a list of strings, we simply create the following class:

```c#
[ComplexType]
public class PersistableStringCollection : PersistableScalarCollection<string>
{
	protected override string ConvertSingleValueToRuntime(string rawValue)
	{
		return rawValue;
	}

	protected override string ConvertSingleValueToPersistable(string value)
	{
		return value.ToString();
	}
}
```

### List of Ints

Similarly, if we need to serialize a list of ints, we simply create the following class:

```c#
[ComplexType]
public class PersistableIntCollection : PersistableScalarCollection<int>
{
	protected override int ConvertSingleValueToRuntime(string rawValue)
	{
		return int.Parse(rawValue);
	}

	protected override string ConvertSingleValueToPersistable(int value)
	{
		return value.ToString();
	}
}
```

That's it, once again thanks Bernhard for this little gem.
