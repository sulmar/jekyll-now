---
layout: post
title: Konwertery w EF Core 2.1
---

# Przykład konwertera EF Core 2.1

## Wstęp
Czasami sposób zapisu wartości jakiejś właściwości w bazie danych różni się od tej zdefiniowanej w klasie. 
Na przykład w modelu posiadamy np. płeć jako enum a w bazie danych chcemy zapisać jako "M" lub "F". 
Albo bardziej złożony przykład - po stronie modelu posiadamy obiekt z adresem lub parametrami urządzenia, a w bazie danych chcemy zapisać go w jednej kolumnie jako xml lub json. 

Niestety w poprzedniej wersji EF 6 nie było gotowego mechanizmu i trzeba było stosować obejścia. 
Najczęściej obejście polegało na tym, że trzeba było utworzyć w klasie dodatkowe ukryte prywatne pole (tzw. backfield) odpowiadające typowi w bazie danych, a docelowa właściwość była oznaczona jako ignorowana przez EF. Następnie w metodach get i set była realizowana konwersja. Nie było to optymalne rozwiązanie i kod nie był przenośny.

W EF Core wprowadzono nową funkcję, tzw. konwertery (ValueConverters), które rozwiązują ten problem w bardzo elegancki sposób. 
Nie trzeba już tworzyć dodatkowych pól, ale przede wszystkim można wielokrotnie używać tej samej konwersji. 
Konwerter można użyć w konfiguracji oraz w konwencji.


## Konwersja za pomocą wyrażeń lambda

~~~ csharp
builder.Property(p=>p.Gender)
  .HasConversion(
      v => v.ToString(),
      v => (Gender)Enum.Parse(typeof(Gender), v)
);
~~~

## Konwersja za pomocą obiektu konwertera
~~~ csharp            
var converter = new ValueConverter<Gender, string>(
v => v.ToString(),
v => (Gender)Enum.Parse(typeof(Gender), v));
~~~

## Użycie wbudowanego konwertera
   
~~~ csharp              
            var converter = new EnumToStringConverter<Gender>();
            builder.Property(p=>p.Gender)
              .HasConversion(converter);
~~~

Niektóre z konwerterów posiadają dodatkowe parametry:
~~~ csharp              
            builder.Property(p=>p.IsDeleted)
                .HasConversion(new BoolToStringConverter("Y", "N"));
~~~

Lista wbudowanych konwerterów [https://docs.microsoft.com/en-us/ef/core/modeling/value-conversions]

                

## Predefiniowane konwersje
W większości przypadków nie trzeba tworzyć konwerterów, bo wystarczy skorzystać z predefiniowanych konwersji:
~~~ csharp 
builder.Property(p=>p.Gender)
                .HasConversion<string>();
~~~                

            
## Własny konwerter

### Własny konwerter za pomocą wyrażenia lambda

~~~ csharp 
  builder.Property(p => p.ShippingAddress).HasConversion(
    v => JsonConvert.SerializeObject(v),
    v => JsonConvert.DeserializeObject<Address>(v));
~~~
   
### Utworzenie własnego konwertera

Utworzenie klasy własnego konwertera
~~~ csharp 
public class JsonValueConverter<T> : ValueConverter<T, string>
    {
        public JsonValueConverter(ConverterMappingHints mappingHints = null)
        : base(v => JsonConvert.SerializeObject(v), 
               v => JsonConvert.DeserializeObject<T>(v), 
               mappingHints)
        {
        }      
    }
~~~

Użycie własnego konwertera
~~~ csharp 
builder.Property(p => p.ShippingAddress).HasConversion(new JsonValueConverter<Address>());
~~~

W celu ułatwienia korzystania z konwertera można utworzyć metodę rozszerzającą

~~~ csharp 
public static class PropertyBuilderExtensions
    {
        public static PropertyBuilder<T> HasJsonValueConversion<T>(this PropertyBuilder<T> propertyBuilder) where T : class
        {
            propertyBuilder
              .HasConversion(new JsonValueConverter<T>());

            return propertyBuilder;
        }
    }
    
~~~

A następnie użyć jej podczas konfiguracji
~~~ csharp 
            builder.Property(p => p.ShippingAddress)
                .HasJsonValueConversion();
~~~

## Dokumentacja
- https://stackoverflow.com/questions/50727860/ef-core-2-1-hasconversion-on-all-properties-of-type-datetime
- https://docs.microsoft.com/en-us/ef/core/modeling/value-conversions

## Linki
https://github.com/Innofactor/EfCoreJsonValueConverter
