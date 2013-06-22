---
layout: post
title: "Nbuilder disable property naming for all fields with given Name"
date: 2013-06-21 20:39
comments: true
categories: 
---
[NBuilder][] is fluent objects builder, generates values for properties/fields in the object automatically. This also to persist the created object using the `.Persist()`.
Citing the example from [NBuilder Persistence][]

{% codeblock Persistence lang:csharp %}
	// in config section
	BuilderSetup.SetCreatePersistenceMethod<Product>(productRepository.Create);
	// in test code
	Builder<Product>.CreateNew().Persist();
{% endcodeblock %}

If NHibernate is being used for persistence and identity insert is off, the NBuilder needs to be told to not generate values for `Id`. This can be done like `BuilderSetup.DisablePropertyNamingFor<Product, int>(x => x.Id);`

However this needs to be done for all the domain classes, and its not straight forward as current way of disabling property naming is only through generic method.

To disable property naming for a property of given name, the following snippet can be used

{% codeblock Disable naming for all fields with given name lang:csharp %}
	typeof(Product).Assembly.GetTypes().Where(HasId).ForEach(t =>
                {
                    var methodInfo = typeof(BuilderSetup).GetMethod("DisablePropertyNamingFor");
                    var idType = t.GetProperty("Id").PropertyType;
                    var disablePropertyNaming = methodInfo.MakeGenericMethod(t, idType);
                    var parameterExpression = Expression.Parameter(t, "t");
                    var makeMemberAccess = Expression.MakeMemberAccess(parameterExpression, t.GetProperty("Id"));
                    var memberAccessExpression = Expression.Lambda(makeMemberAccess, parameterExpression);
                    disablePropertyNaming.Invoke(null, new object[] { memberAccessExpression });
                })


    private static bool HasId(Type t)
    {
        return t.Namespace != null && t.Namespace.Contains("Model") && t.GetProperty("Id") != null;
    }

{% endcodeblock %}

This snippet invokes `BuilderSetup.DisablePropertyNamingFor` with `x=>x.Id` being passed in as a lambda expression doing MemberAccess for all classes in the assembly which has an Id.

[NBuilder]: http://nbuilder.org/ "NBuilder"
[NBuilder Persistence]: http://nbuilder.org/Documentation/Persistence