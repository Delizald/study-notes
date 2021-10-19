## Record types

Part of C# 9.

While records can be mutable, they are primarily intended for supporting immutable data models.
Records are distinct from classes in that record types use value-based equality.

Record types are equal if the record type definitions are identical, and if for every field, the values in both records are equal.

Classes are equal if the objects referred to are the same class type and the variables refer to the same object.

 You define a *`record`* by declaring a type with the `record` keyword.  
you can declare a `record class` to clarify that it's a **reference** type.
You can define a `record struct` to create a record that is a **value** type.

### Value equality

 - Value equality means that two variables of a record type are equal if the types match and all property and field values match.
 - List item

For other reference types, equality means identity. That is, two variables of a reference type are equal if they refer to the same object.

    public record Person(string FirstName, string LastName, string[] PhoneNumbers);
    
    public static void Main()
    {
        var phoneNumbers = new string[2];
        Person person1 = new("Nancy", "Davolio", phoneNumbers);
        Person person2 = new("Nancy", "Davolio", phoneNumbers);
        Console.WriteLine(person1 == person2); // output: True
    
        person1.PhoneNumbers[0] = "555-1234";
        Console.WriteLine(person1 == person2); // output: True
    
        Console.WriteLine(ReferenceEquals(person1, person2)); // output: False
    }
Having this functionality built-in prevents bugs that would result from forgetting to update custom override code when properties or fields are added or changed.

### Nondestructive mutation

you can use a `with` expression to achieve _nondestructive mutation_. A `with` expression makes a new record instance that is a copy of an existing record instance, with specified properties and fields modified. You use object initializer syntax ( {} )to specify the values to be changed, as shown in the following example:

   
    
    public static void Main()
    {
        Person person1 = new("Nancy", "Davolio") { PhoneNumbers = new string[1] };
        Console.WriteLine(person1);
        // output: Person { FirstName = Nancy, LastName = Davolio, PhoneNumbers = System.String[] }
    
        Person person2 = person1 with { FirstName = "John" };
        Console.WriteLine(person2);
        // output: Person { FirstName = John, LastName = Davolio, PhoneNumbers = System.String[] }
        Console.WriteLine(person1 == person2); // output: False
    
        person2 = person1 with { PhoneNumbers = new string[1] };
        Console.WriteLine(person2);
        // output: Person { FirstName = Nancy, LastName = Davolio, PhoneNumbers = System.String[] }
        Console.WriteLine(person1 == person2); // output: False
    
        person2 = person1 with { };
        Console.WriteLine(person1 == person2); // output: True
    }

### Inheritance

A record can inherit from another record. However, a record can't inherit from a class, and a class can't inherit from a record.

##   Init only setters
The `init` accessor makes immutable objects more flexible by allowing the caller to mutate the members during the act of construction. That means the object's immutable properties can participate in object initializers and thus removes the need for all constructor boilerplate in the type.

    public struct WeatherObservation
    {
        public DateTime RecordedAt { get; init; }
        public decimal TemperatureInCelsius { get; init; }
        public decimal PressureInMillibars { get; init; }
    
        public override string ToString() =>
            $"At {RecordedAt:h:mm tt} on {RecordedAt:M/d/yyyy}: " +
            $"Temp = {TemperatureInCelsius}, with {PressureInMillibars} pressure";
    }

Callers can use property initializer syntax to set the values, while still preserving the immutability:

    var now = new WeatherObservation 
    { 
        RecordedAt = DateTime.Now, 
        TemperatureInCelsius = 20, 
        PressureInMillibars = 998.0m 
    };
    
An attempt to change an observation after initialization results in a compiler error:

    // Error! CS8852.
    now.TemperatureInCelsius = 18;

## Extension methods

Extension methods enable you to "add" methods to existing types without creating a new derived type, recompiling, or otherwise modifying the original type. Extension methods are static methods, but they're called as if they were instance methods on the extended type. The parameter is preceded by the this modifier. Extension methods are only in scope when you explicitly import the namespace into your source code with a using directive.  

  namespace ExtensionMethods
  {
      public static class MyExtensions
      {
          public static int WordCount(this String str)
          {
              return str.Split(new char[] { ' ', '.', '?' },
                              StringSplitOptions.RemoveEmptyEntries).Length;
          }
      }
  }

# Lambda Expressions

You use a lambda expression to create an anonymous function.

Expression lambda that has an expression as its body:
  (input-parameters) => expression

Statement lambda that has a statement block as its body:
  (input-parameters) => { <sequence-of-statements> }

Any lambda expression can be converted to a delegate type. The delegate type to which a lambda expression can be converted is defined by the types of its parameters and return value. If a lambda expression doesn't return a value, it can be converted to one of the Action delegate types;

// delegate type
Func<int, int> square = x => x * x;
Console.WriteLine(square(5));
// Output:
// 25

// expression tree 
System.Linq.Expressions.Expression<Func<int, int>> e = x => x * x;
Console.WriteLine(e);
// Output:
// x => (x * x)


## readonly vs const

A const field can only be initialized at the declaration of the field. 
A readonly field can be assigned multiple times in the field declaration and in any constructor.
readonly fields can have different values depending on the constructor used.
a const field is a compile-time constant, the readonly field can be used for run-time constants as in the following example

A readonly field can be assigned and reassigned multiple times within the field declaration and constructor.

A readonly field can't be assigned after the constructor exits. This rule has different implications for value types and reference types:

    Because value types directly contain their data, a field that is a readonly value type is immutable.
    Because reference types contain a reference to their data, a field that is a readonly reference type must always refer to the same object. That object isn't immutable. The readonly modifier prevents the field from being replaced by a different instance of the reference type. However, the modifier doesn't prevent the instance data of the field from being modified through the read-only field.