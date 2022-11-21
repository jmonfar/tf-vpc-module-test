# tf-vpc-module-test

DevOps - Terraform Quiz

## Introduction

In my current work I usually implement infrastructure from modules, not modules themselves, so I don't have yet an existing standard or norm to develop them. Our company has a organization in charge to standardize and adapt (if needed, but sometimes even if not...) the modules to use on our projects, and a 'project skeleton' from which to use them.

I could have taken two approaches to develop the Quiz module:
* Start from empty, writing the module from zero, implementing all resources asked for, and adding later all other not named resources that are required to make the others work.
* Start from an already written and proven module, and adapt it to stick the most strictly possible to the Quiz specifications.

The Quiz redaction could suggest that the first approach is favored, but I always prefer the second. I don't pretend to reinvent the wheel. Almost always someone has found the same issue before and has developed, and often perfected, something to solve it. I prefer to spend effort to improve the existing solution, if pertinent, and contribute back, than to climb again the same hill.

In general, my usual approach is to investigate if there is a good candidate to be adapted, and the amount of effort involved in doing it. Sometimes, at the end, it's less effort to build something from the ground than reforming any existing model (in my experience, though, this very exceptional for anything significantly complex).

In this case, there is a very obvious candidate, the community [terraform-aws-vpc](https://github.com/terraform-aws-modules/terraform-aws-vpc/tree/master/examples) module. I can be confident that it does all that is required here and beyond, and that is done according to the best practices to which I will be measured. It is unlikely that I find anything better.

There are a pair of drawbacks, though:
* I am aware that using a module already built according to best practices does not show that much about my own knowledge and adherence to them. Being able to discern and select the best candidate says something, but not that much as building it all.
* The selected candidate does have so many options and features over the Quiz requirements that requires a heavy modification work to 'reduce' it to fit properly.

Fortunately, I think that both issues do not add, but provide and opportunity to show my skill being able to so heavily 'slim' the module while keeping it working, clean, coherent, and fitting the specifications of the quiz and the best practices. It has been quite a task, but I am still convinced that less work and safer than trying to build from zero.

## Modifying the [terraform-aws-vpc](https://github.com/terraform-aws-modules/terraform-aws-vpc/tree/master/examples) module

The community module main.tf has so many options and features that counts 1251 lines, managing up to 70 different resource types. After adaptation, the module has gone down to 351 lines and 19 resources. Hope its understood that being able to do so implies a quite detailed comprehension of the code and the associated AWS resources, to be able to decide what was droppable and how.

I could have reduced it further, but opted to keep the options that, in my best judgement, were possibily still interesting for the potential module users, or needed to adapt to what the Quiz did not specify concretely. For example, custom tags for the different resources, or many options for VPC and subnets which required values were not specificed in the Quiz.

Cleaning up and adapting the variables.tf, outputs.tf and original module README.md (now [README-usage.md](README-usage.md)) has implied also quite a significant amount of work. After all the changes to the module itself, the variables and outputs needed a careful review to drop what was removed, keep what was kept, and add the few elements that have been added. Hope it's appreciated as it shows how I value details, cleanliness, and overall coherence.

There was one requirement on the Quiz that required going a little further and modify slightly some of the code. From the "Handle the Availability Zones (AZ) according to the number of private/public
networks or vice-versa" requirement I interpret that the module should manage by itself the selection of AZs according to the number of networks provided.

The original module simply received a list of AZs to use, that could be coherent or not with the number of networks created. It could be debatable if that still fit the requirement as is, but I preferred to show more that mostly delete or keep code, and changed the input: now the module gets as entry the region to deploy to (this is unavoidable, even if only by taking a default value), and builds internally in a local the list of AZ to use according to the number of networks to create (1, 2 or 3). Inputs and outputs have been adapted to this, so is the module [README-usage.md](README-usage.md).

I was tempted to consolidate the list of networks to create from two (public and private) to a single one, but that would have implied to synthetically derive the cidr ranges of one set of networks from the other. Without having any clear indication or idea of the final usage of the module, thought that keeping the ability to select all the cidrs was a safer bet.

On the same line, a more detailed usage scenario would open further opportunities for simplification by dropping options or tags that would not get use.

## Comments on code

The code is not heavily commented, at least to my usual standards. I tend to put more of them, always thinking on other people that could be looking or modifyin it later. Not simply to be verbose, but thinking on providing an context of any choices that have required extensive thinking, investigation or trial and error (the kind of things that often are not really obvious).

So don't take the current code as representative of my commenting habits. It's on the terse side.I'm usually in the verbose side, as this text likely shows. But I normally consider the audience of each communication and try to adapt it.

## Usage of `for_each` / `count` / `dynamic` blocks

The original code was already doing extensive usage of `code`. I simply kept it, even if having to modify most usage by removing the deleted options and flags. Hope that shows that I understand how it works.

I could have forced to introduce some `dynamic` / `for_each` on some resource, as a 'demo' of being able to use them. Having already spent a quite significant time with the Quiz, I did not spend more looking at depth where could I put that without looking forced. I think that it is as important to be able to use them so as to be able to choose where it's appropriate, and I consider that the code it is pretty good and appropriate with the `code` usage.

The usage of a `for` expression loop in the local creation of the `azs` variable is the only somewhat interesting addition.

## Testing
The module has been tested to pass `terraform verify` and `terraform plan`. Apply has not been tested to not spend real money on the Quiz, but I'm confident it would work as expected and according to specifications.

Code in [test/](test) folder has been used as example for the test. Results for the plans using one, two or three subnets is kept in the `test-?az.tfplan` files, which can be examined with a `terraform show` command (they would create 18, 26 and 34 resources, respectively).

## CI Pipeline
Have not spent time on playing with CI pipelines for the module. The original module provides a interesting example of github workflows under the .github folder, but trying to adapt it to my module would have required time that could have put the deadline at risk. It is something I would really like to play with, as in my current work I have not much usage of that, only very basic gitlab pipelines with linters on some projects.