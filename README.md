# Findings during code refactoring
> I could write a lot of thing for each and every BookingRepository
> method as well as some BookingController method. However, I
> would just highlight a few things for now.
>
1. `BookingRepository` is quite generic name for a repository. 
What kind of bookings it works with? Does it saves 
Flights, Jobs, Orders or what? I would not be able to 
understand from it's name unless I see/go through all of
it's methods. I would rather create a specific repository for 
each specific type to avoid confusions. e.g 
JobRepository to work with jobs.

2. I see the controller does interact with database. e.g 
Queries job data in `distanceFeed` method of `BookingController`. 
It should  just pass the relevant information required by repository
to query the database.

3. Laravel already provides a way of FormRequest
to validate incoming request so validating them inside
the Controller itself is just not making use of one it's 
great feature.

4. All methods inside `BookingRepository` are so hard
to read and understand due to their length and IF-ELSE hell.
Definitely none of the methods are worth approving during
PR review. 

5. The `getUsersJobs` method of `BookingRepository`
causes `n+1` SQL problem.
# Violation Of SOLID Principle
1. The "S"(single responsibility) says:
> A class should have one, and only one, reason to change.

This so obvious that even if a junior developer would just read the definition
and then go through this code would definitely find it. e.g
What's the purpose of a controller? Should it pass the data 
to the relevant models/repositories or should it also interact
with database? The `BookingRepository` also has a method 
`userLoginFailed` which clearly is something related to
authentication and has nothing to do with `BookingRepository`
itself.

2. The "O" (Open/Closed Principle) in SOLID says: 
> an entity should be open for extensions but 
close for modification.
 
It's almost violated in many
places, but I would give an example of `getPotentialJobIdsWithUserId`
method inside `BookingRepository`. This method would definitely
change in the future again many times everytime we add/remove a 
translator type which clearly violates the **Open for extension
and close for modification principle.**

3. The "L"(Liskove substitution) in SOLID says:
> Ability to replace any instance of a parent class with any
> an instance of it's child class.
> 
I do not see any violation of this at the moment since we
just have a few classes.

4. The "I"(Inversion of Control) in SOLID says: 
>Objects do not create other objects
on which they rely to do their work.

This is also  violated clearly as the `Logger`, `FirePHPHandler` and `StreamHandler`
is initialized in `BookingRepository` but rather injected. This
tightly couples `Logger` and other classes.

5. The "D"(Depedency inversion principle) says: 
> High-level modules should not depend on low-level modules. Both should depend on abstractions.

The principle is also violated because 
`BookingRepository` is tightly coupled to `Monolog\Logger` so
if any dependency in the Logger constructor changes, it would clearly
break our application.  In order to decouple, we should inject 
an interface which is implemented by `Monolog\Logger`. This
would also avoid any issues in the future if the default Logger
for Laravel changes from Monolog to anything else.



