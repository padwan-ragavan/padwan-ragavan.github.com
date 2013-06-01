---
layout: post
title: "Linq To Sql for Custom Methods/Properties - LinqExtensionMethods"
date: 2013-05-31 20:47
comments: true
categories: 
---
If we have a domain like Item below,

{% codeblock lang:csharp %}
    public class Item
    {
        public virtual int Id { get; set; }
        public virtual string Source { get; set; }
        public virtual string Dest { get; set; }
        public virtual bool IsExport { get { return Source != Dest; } }
    }
{% endcodeblock %}

it would be good if we can write a linq query like ` session.Query<Item>().Where(i => i.IsExport).ToList() `,
however we cant as linq to sql doesnt know how to generate sql for IsExport.

To help linq generate sql for our custom properties we can write our own ILinqToHqlGeneratorsRegistry, something like the one below, which is a generic one and works not just for one property but more.

The idea is to expose the the logic in property as expression which can be used to generate sql.

{% codeblock lang:csharp %}
    public class Item
    {
        public static readonly Expression<Func<Item, bool>> IsExportExpression = o => o.Source != o.Dest; 
        public virtual int Id { get; set; }
        public virtual string Source { get; set; }
        public virtual string Dest { get; set; }
        public virtual bool IsExport { get { return this.FromExpression(IsExportExpression); } }
    }

    public static class ExpressionExtensions
    {
        public static TReturn FromExpression<T, TReturn>(this T o, Expression<Func<T, TReturn>> selector)
        {
            return selector == null ? default(TReturn) : selector.Compile()(o);
        }
    }

    public class LinqToHqlGenerationRegistry : DefaultLinqToHqlGeneratorsRegistry
    {
        public LinqToHqlGenerationRegistry():base()
        {
            RegisterGenerator(ReflectionHelper.GetMethodDefinition(()=>ExpressionExtensions.FromExpression<object,object>(null,null)), new HqlFromExpressionGenerator());
        }
    }

    class HqlFromExpressionGenerator : BaseHqlGeneratorForMethod
    {
        public HqlFromExpressionGenerator()
        {
            SupportedMethods = new[]
                {
                    ReflectionHelper.GetMethodDefinition((() => ExpressionProjectionExtensions.FromExpression<object, object>(null, null)))
                };
        }

        public override HqlTreeNode BuildHql(MethodInfo method, Expression targetObject, ReadOnlyCollection<Expression> arguments,
                                             HqlTreeBuilder treeBuilder, IHqlExpressionVisitor visitor)
        {
            var expression = arguments[1] as UnaryExpression;
            dynamic operand = expression.Operand;
            var replace = ReplacingExpressionTreeVisitor.Replace(operand.Parameters[0], arguments[0], operand.Body);
            return visitor.Visit(replace);
        }
    }

{% endcodeblock %}

we have to add `LinqToHqlGenerationRegistry` to NHibernate using 
```
Fluently
.Configure(yourConfig)
.ExposeConfiguration(config=>config.LinqToHqlGeneratorsRegistry<LinqToHqlGenerationRegistry>());
```

now when we want to query Item we can do 

```
	session.Query<Item>().Where(i => i.FromExpression(Item.IsExportExpression)).ToList()
```

Properties can now share expessions between them.