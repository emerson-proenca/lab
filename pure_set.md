## The Search for a "Pure" Language

I have found that no previous language is considered **PURE** by my definition: The only primitives are **Sets** and/or **Tuples**.

### SETL

Even **SETL** "failed" at this:

> "Every language has certain basic entities which can be manipulated. In SETL these entities are 'atoms', 'sets' and 'tuples'.
> SETL atoms include most of the elementary data types found in other languages: integers, reals, Boolean values, bit strings, character strings, labels, subroutines, and functions. In addition, there are two special types of atoms which play an important role in the language: blank atoms and the undefined atom."

By my definition, it is not the same. It is a high-level language compiling down to types, not "pure." Regarding the paper *"Objects of Alternative Set Theory in Set@l Programming Language,"* the name "Set@l" is likely a stylistic choice or a specific implementation variant, possibly related to **SETL2**.

Source: https://www.researchgate.net/publication/334827393_Objects_of_Alternative_Set_Theory_in_Setl_Programming_Language
Source: [LINK_TOO_LONG](https://pdf.sciencedirectassets.com/271503/1-s2.0-S0898122199X00034/1-s2.0-0898122175900115/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjECIaCXVzLWVhc3QtMSJGMEQCIGGCUy%2BCPeGw8a%2F%2F5EH1FUX9p%2BZBCyCcYJtu64OgyxRdAiB0AbbnSz5x2NF1U7yruLwLuulC0XStuE7CpfsyVnuQFiq8BQjr%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAUaDDA1OTAwMzU0Njg2NSIMqf3HRNBIqGN9iHguKpAFDxUGjR%2B4C7F3HqSj6tMb43opD9oIDpq38cY7P4C47o8OChpdSjmLEm2iCc47ljrGtN6nW%2Br6Lrxrf%2BwrtiEAwsymoCu2x79WZ94Oqw%2Fygqx94s7FfXi1gBZh181%2BnseJKNd7WiaNmCYByFOfiFikjpPOLOtBD%2Bqc%2FDrQgjP228Vx3ImP7PRTqXjGgbzt11n4QugjkysUu6ECsoxI2ojmorL6jMIWm3bl19gysl71es7ThPlnensmd0b0T%2F7bd69j%2BzoKw08rPYOAWR4whzFpTs9hvJq9W2s05StC6TOPYUo8gL4%2Bnt4fZVa%2Bp5eeZP6Fr7lzXDRvWuCsZGySDOorxvpnBbnTJdmUaX4NvEJyEhKm0ULZQCVR0AutftC8NV9nxDJCj1MV4BFYWtOpJxJ05iSsxhUH2uLZDle7U8TAhc7Bpmq3yQAHyJOsUbOyn0p61I%2FDVa5QYkz4cBIoPM%2F1kozdyZkxFUKl%2FTHYUOTNE1o8T2siR%2F1vL%2F%2Fhj1MC0zjCmcCvcIBXfX3%2Fjea%2BFiVqRZJCDur%2BMavHzOyhdmp2b9RV2tDaOCjzipOoI98xZ1259w3wecEDzBV2UV39ysonxCV0f1DRe1Ic2Vwx0T0T6EUrVfIWQw%2B2yE23tReAD9kJVUr3Uustdfp%2BvrQ6peiKqOOeH0KBpCfDIHoPjcISelNyNS0DKT2aRvSUMkWSR4WrlHQ8J%2BLEeqgqvCYjL8DHzy3E6z46YSP%2Fcudg92P8S2dOElKU8ji6X5xr2dPJ9a515bFQEzQvRPZJPpzc%2Fizptdd4EdpW6Cn8R%2BsNfgvBMOv0mlyHmLbuYH1pTzJ%2BJqF27zBQLJPgexrsMombsIBZYa%2BsZPQpQpPkUYaozngWY%2FEwp8aDzAY6sgHHCuO1fXSVcJV8utA8FDwxgTK2ME3ob86N5fX%2BPw%2B8NetTQhCDcn6WrWyitYqeKvhV54g2k1zkgt7N99deD1up8b154j6%2Fes1KXDzsaqGYDpu9FjBFvyGb%2BXPHnlVj%2FLWWAzMNGsBNVE2QteHpxvtolC7oLriHA43KCkYXrbRfcjyhoN1SpHF8VY51PJsHbkqBzV5OqdaxX%2FG6SoDv7cobFbgdaVBFR9oMS7VgENqJY2TV&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20260202T183311Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTYWWT7UAJI%2F20260202%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=6571229af855377c1d1c6fe4594d131b5f9ae707436d250e8d0e50af9cce724a&hash=315a4f1ea2d5d95fd3ed91c7b6a69d88aea4f40cb8081ba834d91b0ee07d9d06&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=0898122175900115&tid=spdf-bcde7a4c-da88-4a2c-a94c-f09d31e36237&sid=bf70693d81f015464a680003ba0e4042bc24gxrqa&type=client&tsoh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&rh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&ua=06155705550102585b&rr=9c7bc6843c04b3a6&cc=br)

---

### Miranda

**Miranda** also relies on predefined types:

> "The primitive types, in addition to **num**, are **bool** (comprising True and False) and **char**, which is the ASCII character set... We now discuss the facilities available in Miranda for user-defined types. There are three mechanisms: (i) type synonyms, (ii) algebraic types, (iii) abstract types."

Source: https://www.cs.kent.ac.uk/people/staff/dat/miranda/nancypaper.pdf

---

### Claire

**Claire** treats types as sets, but maintains them as distinct primitives:

> "Although primitive data types (such as string or list) may be considered as sets, they are seen as infinite sets... New sets can be formed through selection or set image."

**Example Syntax:**

```claire
{x in Person | x.age > 18} 
exists(p in {x.father | x in (man U woman)} | p.age < 20) 
list{x.age | x in Person} 
for x in {f(y) | y in (1 .. 10)} print(x) 
sum(list{y.salary | y in {y in Person | y.dept = sales}}) 

```

Source: https://sites.google.com/view/claire4/home

---

### LINQ

**LINQ** is heavily typed and behaves more like C#, Java than expected:

```csharp
[Table(Name="Customers")]
public class Customer
{
     [Column(IsPrimaryKey = true)]
     public int CustID;

     [Column]
     public string CustName;
}

```

Source: https://learn.microsoft.com/en-us/archive/msdn-magazine/2008/august/basic-instincts-increase-linq-query-performance

---

### Bandicoot

**Bandicoot** includes standard type systems and immutability:

> "Functions are the primary means of accessing and changing data in Bandicoot... The type of a temporary variable is determined from the value assigned to it and the value is immutable."

Source: https://bandilab.github.io/getting-started.html
GitHub: https://github.com/bandilab/bandicoot

---

### Summary

SQL and MATLAB also rely heavily on built-in types. It appears I am still in new territory. So far, no language has achieved:

**Data && Types && Functions && Code == Enum (Set, Tuple)**
