---
layout: post
title: genetic-sort
permalink: /:title
---

A friend of mine is working on a course scheduling algorithm for the university
we attended. The problem is NP-Complete, but through some clever optimizations
and dynamic programming he was able to generate passing schedules... sometimes.
Apparently the order that he applies the scheduling constraints seems to randomly
affect the quality of the solution. Since he couldn't find a pattern to how
the order of application of constraints affected the outcome of his algorithm,
he decided to use a genetic algorithm to search for an an optimal permutation.

<!-- more --> 
 
Since neither of us is that experienced with genetic algorithms, I figured I could
do a little proof of concept in Python. His constraint data structures are
pretty deeply embedded in his C++ codebase, so I decided to solve a different
problem with a similar structure - sorting. While this may be one of the worst
practical applications of genetic algorithms in existence, it does follow the
same pattern of searching for a permutation to maximize a fitness function.
 
If you slept through your undergraduate AI lectures, here's nice [primer](http://www.ai-junkie.com/ga/intro/gat1.html)
on genetic algorithms. It roughly boils down to these steps:
 
1. Write a fitness function to score a solution based on how good it is.
2. Write a crossover function to combine two solutions into a new, unique solution.
3. Generate a sample population of random solutions.
4. Randomly select two solutions (with a bias towards better-fit solutions).
5. Crossover (breed) those two solutions into a new solution and add it to the population.
6. Every once in awhile, introduce a random mutation (as is done in nature).
7. Repeat 4:6 until an optimal solution is born
 
Realistically, there's probably a lot more caveats and considerations depending on
the type of problem you're dealing with, but this is a POC in Python so let's just
do it.
 
### Implementing the parts
 
Let's start with a fitness function to determine "how sorted" a string is.
This works by counting how many characters in the string are 1 less than
that follows it.
 
{% highlight python %}
def fitness(s):
    return sum(ord(s[i]) == ord(s[i+1])-1 for i in range(len(s)-1))
{% endhighlight %}
 
For the sample population, we're going to use random permutations of the alphabet.
Let's also sort them by fitness (yes, we are using sorting to implement sorting).
 
{% highlight python %}
alphabet = np.array(list('abcdefghijklmnopqrstuvwxyz'))
population = np.array([permutation(alphabet) for _ in range(SAMPLE_SIZE)])
population = np.array(sorted(population, key=fitness))
{% endhighlight %}
 
Next we need a function for selecting two solutions from the population
to crossover. Since we want to select with a bias towards good solutions,
we can use a probability distribution relative to the fitness of each solution.
 
{% highlight python %}
def select(population):
    p_dist = np.fromiter(map(fitness, population), dtype=np.float)
    p_dist /= np.sum(p_dist)
    idx = choice(len(population), 2, p=p_dist)
    return population[idx]
{% endhighlight %}
 
Now we need a way to combine two solutions into a new one. This is called
a "crossover." Depending on the structure of the solution, there's quite a
few well known methods for crossing them together. The criteria of this
crossover function is that it maintains all the unique characters from
the string while still inheriting some "traits" of the parent strings.
For this we can use an [Ordered Crossover](https://stackoverflow.com/a/26521576/2945912).
 
{% highlight python %}
def crossover(p1,p2):
    i = randint(0,len(p1))
    j = randint(i,len(p1))
    child = p1[i:j]
    remain = list(filter(lambda c: c not in child, p2))
    child = np.concatenate((remain[:i], child, remain[i:]), axis=0)
    return child
{% endhighlight %}
 
And finally, we need a way to introduce a small mutation every so often.
This should help prevent the search from getting stuck at some local
optimum.
 
{% highlight python %}
def mutate(s,rate):
    if random() < rate:
        for _ in range(5)
          i,j = randint(0,len(s)),randint(0,len(s))
          s[i],s[j] = s[j],s[i]
    return s
{% endhighlight %}
 
### Bringing it together
 
Now that we have all the parts, were ready to assemble this genetic algorithm:
 
{% highlight python %}
for _ in range(GENERATIONS):
    p1,p2 = select(population)
    child = crossover(p1,p2)
    child = mutate(child,MUTATION_RATE)
    insort_index = bisect_left(list(map(fitness, population)), fitness(child))
    population = np.insert(population, insort_index, child, axis=0)
    population = np.delete(population, 0, 0)
{% endhighlight %}
 
One small optimization I added was to keep the population size constant. That way
we can simulate as many generations as needed without blowing up the amount of
computation required for each generation.
 
### Results
 
Here's the results after running it for a few generations.
 
```
[GENERATION 1000] Most fit: zcdajmbklqriopvwxghstuynef (fitness=10)
[GENERATION 2000] Most fit: lmnjkqrdvwxyzabcghopstuefi (fitness=15)
[GENERATION 3000] Most fit: lmnkjdefvwxyzabcghopqrstui (fitness=17)
[GENERATION 4000] Most fit: lmnjkdefvwxyzabcghopqrstui (fitness=18)
[GENERATION 5000] Most fit: lmnjkvwxyzabcdefghopqrstui (fitness=20)
[GENERATION 6000] Most fit: lmnjkvwxyzabcdefghopqrstui (fitness=20)
[GENERATION 7000] Most fit: lmnjkvwxyzabcdefghopqrstui (fitness=20)
```
 
After running it a few times it seems work pretty well until it hits the 18-20
fitness range where it stays indefinitely. That's probably because once we
hit that level of sortedness, there are very few crossovers and mutations
that will bring us closer to a perfect sort.
 
What I gathered from this is that the performance of a genetic algorithm
probably has a lot to do with the solution space and the cleverness of
the crossover and fitness functions. With sorting, it seems like the
solution space is littered with steep local optimums that are easy
for the genetic algorithm to get trapped in. Hopefully that won't be
the case with my friend's constraint scheduling algorithm, but if so
then we'll have to find a more suitable optimization approach.
 

