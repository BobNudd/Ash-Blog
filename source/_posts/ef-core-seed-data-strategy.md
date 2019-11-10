---
title: EF Core – Seed Data Strategy
date: 2019/01/06 22:49:00
tags: c#, seed data, .net core
---

In this post, we’re going to explore the *relatively* new entity data seeding method which was introduced with EF Core 2.1.

**It’s fantastic**.

Unlike it’s predecessor EF6, seed data can be associated with migrations and it’s relative entity type as part of your migration strategy. What this means is adding or editing any seed data for any entity will compute the correct insert(s) or updates to push those data changes into your DB, this is incredibly powerful and leverages the benefits of tracking that migrations provide.

### Lets see some code

First, you’ll need to override the `OnModelCreating` method in your DbContext, chances are you’re already doing this in an existing project.

{% codeblock lang:csharp %}
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder)   
    }
{% endcodeblock %}

Next, we’ll call the `HasData` method and pass in a populated instance of an entity.

{% codeblock lang:csharp %}
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        modelBuilder.Entity().HasData(new Person { PersonId = 1, Name = "Ash" });
    }
{% endcodeblock %}

Good so far? Before running our `add-migration` command, I want to touch on a few key points:

* Primary keys for all entities must be specified, this is how the seed data change detection works via comparing if the particular record exist, or if any fields have changed
* Any entities with foreign keys must be explicitly stated
* Previously seeded data that has a primary key change will be removed (this makes sense, since the record can no longer be referred to)
* You may want custom initialization logic depending on your scenario such as temporary data for testing

Now, running `add-migration firstseed` will produce a migration for entering our new records (since their primary keys don’t exist in the table).

{% codeblock lang:csharp %}
migrationBuilder.InsertData(
    table: "Persons",
    columns: new[] { "PersonId", "Name" },
    values: new object[] { 1, "Ash" });
{% endcodeblock %}

Any changes to this data will result in a new migration with correct behaviour (insert/update/delete) making seed data much easier to progress as your applications develops, rather than a monolithic script of inserts.

### Further Points

##### Structuring Seed Data

Adding seed data with the `OnModelCreating` can become difficult to manage, especially with dozens of entities, personally I would opt to create extension methods to keep this method clean.

{% codeblock lang:csharp %}
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        builder.AddContractHoursData();
        builder.AddLocationData();
        builder.AddJobTitleData();
    }
{% endcodeblock %}

Additionally, creating a single contained class for each data entity was my approach, adhering to the Single Responsibility Principle and violating DRY.

{% codeblock lang:csharp %}
    public static void AddRewardCategoryData(this ModelBuilder model)
    {
        model.Entity().HasData(new List()
        {
            new RewardCategory()
            {
                RewardCategoryId = 1,
                Name = "Vouchers",
                Enabled = true
            },
            new RewardCategory()
            {
                RewardCategoryId = 2,
                Name = "Charity",
                Enabled = true
            }
        });
    }
{% endcodeblock %}

##### Creating Identity Users

One question I’ve seen people ask is seeding users from Identity (since `UserManager` takes care of things like Password Hash).

We can use Identities `PasswordHasher`.

However there’s a problem with this approach, this will programmatically generate a different hash per run, and therefore bloat every migration with updates. Additionally you’d rather avoid hard coding dozens or hundreds of type `User` models.

My strategy for this approach was the following:

* Use a library such as [Faker](https://www.nuget.org/packages/Faker.Net/) to generate a list of  user details
* Iterate through the list, assign a hard-coded GUID from a list you’ve created
* Use the `PasswordHasher` to generate the hashed password
* Pass the completed list to the `HasData` method
* Copy the data within the AspNetUser table in your database as JSON (I used SQL Operations Studio for this)
* Use some Find and replace with regular expressions  to fix formatting errors and form into a list of Users
* You should a list of populated entities, all with hard coded data without tedious data entry