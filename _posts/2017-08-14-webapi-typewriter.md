
##  Typewriter and Web Api Part 1 ##

Using Typewriter with Web Api to create Typescript Interfaces Part 1.

I had the following goals for using [TypeWriter](https://github.com/frhagn/Typewriter):

1. Simplify the creation of TypeScript interfaces & classes for calling the back-end API.
2. Standardize the way that the classes are represented.
3. Make it easy to include those classes in my fron-end TypeScript-based application.

I created the folloiwng template for TypeScript. It can:

1. Represent base classes and inheritance. 
2. Change Pascal cassed names into camel case.
3. Change enumerations into a sting type with a list of valid values.
``` csharp 
    ${
        using Typewriter.Extensions.Types;

        bool HasBaseClass(Class c)
        {
            return c.BaseClass != null;
        }
    
        string ExtractedValue(Attribute attribute)
        {
            var index = attribute.Value.IndexOf("=");
            return attribute.Value.Substring(index + 1).Trim();
        }

        string GetTypeName(Type t) 
        {
            // TypeScript types that are Pascal Cased.
            if (t.Name == "Date")
            {
                return t.Name;
            }
            return t.name;
        }

        Template(Settings settings)
        {
            settings.OutputFilenameFactory = file => 
            {
                return $"{file.Name.Substring(0,1).ToLower()}{file.Name.Substring(1,file.Name.Length - 1).Replace(".cs", ".ts")}";
            };
        }
    }

    module beefBooksApi {

        $Classes(*Model)[
        export interface i$Name$TypeParameters $BaseClass[extends i$Name$TypeArguments]{
            $Properties[
            $name: $Type[$GetTypeName];]
        }
    
        export class $name$TypeParameters $BaseClass[extends $name$TypeArguments] implements i$Name$TypeArguments  {
            $Properties[
            $name: $Type[$GetTypeName];]  
    
            constructor() {         
            $HasBaseClass[super();][//nothing]                   
            }
        }]

        $Enums(e=>e.Namespace == "FeedlotManagement.Models" || e.Namespace == "FeedlotManagement.Identity.Models")[
            export type $name = $Values[$Attributes[$ExtractedValue]][ | ];
        ]

        $Classes(e => e.Namespace == "FeedlotManagement.Models" && e.Name == "DateRange")[
            export class $name$TypeParameters { $Properties[
                $name: $Type[$GetTypeName];]
        }]

    }
```

1. A class with its base class converted by Typewriter.

    1.1 First the C#:
```csharp
    namespace FeedlotManagement.Api.Models
    {
        public class BaseModel
        {
            public int Id { get; set; }
        }
    }

    public class ContactModel : BaseModel
    {
        public string EmailAddress { get; set; }

        public string FirstName { get; set; }

        public string LastName { get; set; }

        public Occupation Occupation { get; set; }

        public bool CanLogin { get; set; }

        public Role ApplicationRole { get; set; }

        public string IdentityId { get; set; }

        public ContactModel()
        {
        }

        public ContactModel(Contact contact, ApplicationUserManager userManager)
        {
            this.EmailAddress = contact.EmailAddress;
            this.FirstName = contact.FirstName;
            this.LastName = contact.LastName;
            this.Occupation = contact.Occupation;
            this.CanLogin = !string.IsNullOrEmpty(contact.IdentityId);
            this.ApplicationRole = contact.GetApplicationRole(userManager);
            this.Id = contact.Id;
            this.IdentityId = contact.IdentityId;
        }
    }
```
    1.2 The resulting TypeScript:
```typescript
    export interface iBaseModel {
        id: number;
    }

    export class baseModel implements iBaseModel {
        id: number;

        constructor() {

            //nothing
        }
    }

    export interface iContactModel extends iBaseModel {
        emailAddress: string;
        firstName: string;
        lastName: string;
        occupation: occupation;
        canLogin: boolean;
        applicationRole: role;
        identityId: string;
    }

    export class contactModel extends baseModel implements iContactModel {
        emailAddress: string;
        firstName: string;
        lastName: string;
        occupation: occupation;
        canLogin: boolean;
        applicationRole: role;
        identityId: string;

        constructor() {
            super();
        }
    }
```

2. An enumeration converted by Typewriter

    2.1 First the C#:
```csharp 
    public enum Occupation
    {
        [EnumMember(Value = "Unknown")]
        Unknown = 0,

        [EnumMember(Value = "Butcher")]
        Butcher = 1,

        [EnumMember(Value = "Baker")]
        Baker = 2,

        [EnumMember(Value = "Candlestick Maker")]
        CandlestickMaker = 3
    }
```

    2.2 The resulting TypeScript:
```typescript
        export type occupation = "Unknown" | "Butcher" | "Baker" | "Candlestick Maker";
```

