# Roles Are Just an Abstraction: Rethinking Authorization from First Principles

> _While building an authorization system, I independently arrived at a policy-first way of thinking. After looking existing IAM systems, I realized that many mature authorization platforms already think this way internally. This article isn't about inventing a new authorization model. It's about the engineering thought process that led me there._

***

### Introduction

Every non-trivial application eventually needs authorization.

Whether you're building an LMS, CRM, SaaS platform, internal developer portal, or banking application, you'll eventually need to answer a simple question:

> **"Can this user perform this action on this resource?"**

Most engineers immediately reach for Role-Based Access Control (RBAC), and for good reason. It is simple, intuitive, and works extremely well for many applications.

This article isn't about why RBAC is bad.

It's about how building a real world authorization system made me question some of the assumptions I had about roles.&#x20;

Along the way, I realized I wasn't really solving a "role problem" I was solving a "permission problem."



### Side Note

**Note the problem is not STS credential remain active till it expires this is a know situation the problem is the defences that AWS give you does not work against a admin user**

***

## Starting Simple

To make the discussion concrete, let's imagine we're building an LMS.

Initially the authorization model is straightforward.

We have:

* Student
* Course Creator
* Platform Administrator

Traditional RBAC handles this perfectly.

```
Student
Creator
Administrator
```

Each role maps to a collection of permissions.

At this point, everything is simple.

***

## The First Crack Appears

As the application grows, so do the business requirements.

Now we introduce another role.

**Course Editor**

Unlike creators, editors don't create content. They review existing lessons, fix grammatical mistakes, update metadata, and improve documentation.

Still manageable.

Then another requirement arrives.

* Alice can edit only the AWS course.
* Bob can edit only the CRTO course.
* Charlie can edit both.

Suddenly, the role itself is no longer enough.

A role now needs context.

It needs **scope**.

***

## The Obvious Solution

The first solution most of us think of is simply creating more roles.

```
course_editor_aws
course_editor_crto
course_editor_oscp
```

Technically...

It works.

But something immediately felt wrong.

The role name is no longer describing a responsibility.

It is describing both responsibility and data.

The more courses I create, the more roles I need.

Eventually I realized I wasn't managing permissions anymore.

I was managing role names.

This is the classic problem known as **role explosion**.

***

## Asking The Wrong Question

Initially I kept asking myself:

> "How do I build better roles?"

Eventually I realized that wasn't the right question.

The better question was:

> **"What exactly is a role and what is a permission?"**

That single question completely changed how I thought about authorization.

***

## What Is A Permission?

The more I looked at authorization, the more every decision appeared to answer the same questions.

* Who is making the request?
* What action are they performing?
* Which resource are they acting on?
* Should the request be allowed?

From a database perspective, every permission naturally became something like:

```
principal
resource_type
action
resource_id (optional)
effect
```

Examples:

```
Nathaniel
lesson
update
54
allow
```

or

```
Nathaniel
course
publish
OSCP
deny
```

At this point something interesting happened.

***

## Then What Is A Role?

If permissions describe what someone can actually do...

Then what exactly is a role?

Stepping away from software for a moment and thinking about organizations instead, a role is simply a collection of responsibilities.

Software represents responsibilities as permissions.

Which means...

A role is simply a reusable collection of permissions.

For example:

```
Course Editor
=
lesson.update
lesson.publish
lesson.archive
module.update
```

Nothing more.

That realization changed how I viewed authorization.

***

## Roles Are Not The Primitive

Traditional RBAC is usually represented like this:

```
User
 |
Role
 |
Permission
```

But I started thinking about it differently.

```
User
       \
        \
         Effective Permissions
        /
Role
```

The authorization engine doesn't actually care where permissions came from.

They could come from:

* A role
* A group
* A direct assignment
* A temporary permission
* A future feature that doesn't even exist yet

Eventually everything becomes a permission.

Roles simply become a convenient way for humans to manage collections of permissions.

***

## Separating Storage From Representation

Another realization came from the database design itself.

Developers naturally think about permissions like this:

```
lesson.update
```

But databases don't necessarily need to store permissions that way.

Instead of storing a string, I chose to normalize it.

```
resource_type = lesson
action = update
```

The API can still expose:

```
lesson.update
```

while the database stores structured attributes that are easier to query, validate, index, and analyze.

This was another important realization.

The API representation and the storage representation don't need to be the same thing.

***

## Looking At Existing IAM Systems

After arriving at this model, I started researching how mature authorization systems work.

Many mature authorization systems already think this way.

AWS IAM, Azure RBAC, Google Cloud IAM, and Kubernetes all treat permissions or policies as the fundamental unit of authorization.

Roles are simply reusable collections of those permissions.

I had independently arrived at a similar way of thinking by repeatedly questioning each abstraction instead of accepting it.

***

## What I Learned

The biggest lesson wasn't about authorization.

It was about engineering.

Instead of asking:

> "How should I implement roles?"

I started asking:

> "Why do roles exist?"

That small shift in thinking led me to a much cleaner architecture.

Not because the architecture was new.

But because I finally understood the abstraction instead of simply implementing it.

***

## Where I Go From Here

This article intentionally focused on the thought process rather than the implementation.

***

## Final Thoughts

I no longer think of roles as the foundation of authorization.

I think of permissions as the primitive.

Roles simply exist to make those permissions easier for humans to manage.

More importantly, it changed how I think about authorization itself.



