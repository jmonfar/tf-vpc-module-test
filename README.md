# `DevOps - Terraform Quiz` / tf-vpc-module-test 
## Introduction

In my current job I usually implement infrastructure from modules, not modules themselves. Our company has a organization in charge to select and adapt the modules (if needed, but sometimes even if not really needed...), and a 'project skeleton' from which to use modules and enforce a consistent project structure and standards.

I could have taken two approaches to develop the Quiz module:
* Start from empty, writing the module from zero, implementing all resources explicitly asked for, and adding later all other required to make it all work.
* Start from an already written and proven module, and modify it to stick to the Quiz specifications.

The Quiz redaction could suggest that the first approach is favored, but I always prefer the second for anything significantly complex. I don't want to reinvent the wheel. Almost always someone has found the same issue before and has developed, and often perfected, something to solve it. I prefer to spend effort to improve the existing solution, if pertinent, and contribute back, than to climb again the same hill.

My usual approach is to investigate if there is a good candidate to be adapted, and the amount of effort involved in doing it, to assure it pays up. Exceptionally, sometimes it shows up that it's less effort to build from the ground than reforming what I found.

In this case, there is a very obvious candidate, the community [terraform-aws-vpc](https://github.com/terraform-aws-modules/terraform-aws-vpc/tree/master/examples) module. I can be confident that it does all that is required here and beyond, and doing it according to the best practices to which I will be measured. It is unlikely that I find anything better.

There are a pair of drawbacks, though:
* I am aware that using a module already built according to best practices does not show that much about my own knowledge and adherence to them. Being able to discern and select the best candidate says something, but not as much as building it all.
* The selected candidate does have so many options and features over the Quiz requirements that requires a heavy modification work to 'reduce' it to fit properly.

Fortunately, I think that both issues combine to provide and opportunity to show my skill being able to so heavily 'slim' the module while keeping it working, clean, coherent, and fitting the specifications of the quiz and the best practices.

It has been quite a task, but I am still convinced that it took less work and was safer than trying to build from zero. Without any previous reference, it would have been hard to find all the complementary resources needed for the explicitly requested to work. And would have come with a lot less flexible module.

So for most practic purposes, my module is so different that it cannot be considered the same, but a derivative work for which we have always to credit the original authors.

## Modifying the [terraform-aws-vpc](https://github.com/terraform-aws-modules/terraform-aws-vpc/tree/master/examples) module

The community module `main.tf` has so many options and features that counts 1251 lines, managing up to 70 different resource types. After adaptation, the module has gone down to 351 lines and 19 resources. Hope its understood that being able to do so implies a quite detailed comprehension of the code and the associated AWS resources. Otherwise it would be impossible to discern what was droppable and how to glue the holes between the remaining fragments.

I could have reduced it further, but opted to keep the options that, in my best judgement, were possibily still interesting for the potential module users, or needed to leave open what the Quiz did not specify concretely. For example, custom tags for the different resources, or many options for VPC and subnets with required values that were not specificed in the Quiz.

A more defined usage scenario would open further opportunities for simplification by dropping options or tags that would not get use.

There was one requirement on the Quiz that prompted going a little further and modify some of the code. From the "Handle the Availability Zones (AZ) according to the number of private/public
networks or vice-versa" requirement I interpret that the module should manage by itself the selection of AZs according to the number of networks provided.

The original module simply received a list of AZs to use, that could be coherent or not with the number of networks created. It could be debatable if that already fit the requirement as is, but I opted to change the input: now the module gets as entry the `region` to deploy to (this is unavoidable, needs at least a default value). Then it builds internally in a local variable the list of AZ to use according to the number of networks to create (1, 2 or 3), using a `for` expression loop. Inputs and outputs have been adapted to this, so is the module [README-usage.md](README-usage.md).

I was tempted to consolidate the lists of networks to create from two (public and private) to a single one, but that would have implied to synthetically derive the cidr ranges of one set of networks from the other. Without having any clear indication or idea of the final usage of the module, thought that keeping the ability to select at choice all the cidrs was a safer bet.

Another significant modification over the original module came from the Quiz requirement to always have same number or public and private subnets. The original module did not put any condition on the number, or even existence, of any of them.

In order to assure that the condition is met, first thought on adding conditions to the variable definition, but found the conditions could not refer to any other variable than the one being defined. To solve this, I combined both input variables (`private_subnets` and `public_subnets`) in a single object variable (`subnets`) grouping both lists, so a mutual coherence condition could be added, plus another sanity check to assure that the number of subnets is between 1 and 3.

Cleaning up and adapting the variables.tf, outputs.tf and original module README.md (now [README-usage.md](README-usage.md)) has implied also quite a significant amount of work. After all the changes to the module itself, the variables and outputs needed a careful review to delete what was dropped, adapt what was kept, and add the few elements that have been added. Hope it's appreciated as it shows how I value details, cleanliness, and overall coherence.

## Comments on code

The code is not heavily commented, at least to my usual standards. I tend to put more of them, always thinking on other people that could be looking or modifying it later. Or even myself finding it months or years later and thinking 'what the hell did I do here'. Not simply to be verbose, but for providing a context of any choices that have required extensive thinking, investigation or trial and error (the kind of things that often are not really obvious).

So don't take the current code as representative of my commenting habits. It's on the terse side.I'm usually in the verbose side, as this README clearly shows. But I normally consider the audience of each communication and try to adapt it, and I don't think that for this module more comments would be useful.

## Usage of `for_each` / `count` / `dynamic` blocks

The original code was already doing extensive usage of `code`. I simply kept it, even if having to modify most usage by removing the deleted options and flags. Hope that shows that I understand how it works.

I could have forced to introduce some `dynamic` / `for_each` on some resource, as a 'demo' of being able to use them. Having already spent a quite significant time with the Quiz, I did not spend more looking at depth where could I put that without looking forced. I think that it is so important to be able to use them as being able to choose where it's appropriate. I consider that the code it is pretty good and appropriate with the `code` usage.

The usage of a `for` expression loop in the local creation of the `azs` variable is the only somewhat interesting addition, also with the `subnets` entry and condition refactoring.
## Best practices

A short comment about adherence to best practices: I can't claim merit for the module interface/functionality deepness ratio. My modified module does way less than the original one, and the numever of input variables has been also reduced. For me the important concept is that the module can be used with the bare minimum of required inputs (only the `subnet` entry is really required). All the other are optional to adapt to possible different needs. I think that the input variables are as intuitive as they can for providing complex resources like those, and have sensible defaults (at least with the degree of requirement definition available).

With more defined input requirements, even the `subnet` variable could have some well agreed defaults and be optional. And perhaps de 'eu-west-1' `region` default only makes sense to me, and should be different or not have default at all. Would depend on requirements to be defined.

Other best practices really refer to module lifecycles, what happens after the module is created and used and later evolved and modified. They can't be evaluated for a newly birth module like this. Of course woul take into account to minimize the diffs after modification to the really necessary, with new features off by default, and consider using the new `moved` block to minimize impact.
## Testing
The module has been tested to pass `terraform verify` and `terraform plan`. Apply has not been tested to not spend real money on the Quiz, but I'm confident it would work as expected and according to specifications.

The validation condition for `subnets` variable have also been tested. Providing subnet numbers not between 1 and 3, or not equal between private and public, abort the plan operation with the expected error code.

Code in [test/](test) folder has been used as example for the test. Results for the plans using one, two or three subnets is kept in the `test-?az.tfplan` files, which can be examined with a `terraform show` command (they would create 18, 26 and 34 resources, respectively).

## CI Pipeline
Have not spent time on playing with CI pipelines for the module. The original module provides a interesting example of github workflows under the .github folder, but trying to adapt it to my module would have required time that could have put the deadline at risk. It is something I would really like to play with, as in my current job I have not much usage of that, only very basic gitlab pipelines with linters on some projects.