---
title: "Keep those PRs small!"
date: 2019-12-07T17:30:00+00:00
draft: true
---

*tl;dr: Keeping your pull requests small will lead to quicker and better quality reviews, allowing you to
merge your glorious code into master much sooner.*

![Keep those PRs small!](/images/keep-those-prs-small/main.png)

*Disclaimer: Many of the behaviours described in this post have been exhibited by myself. I am not directing criticisms
towards anyone in particular.*

## Storytime

Picture this. You've just arrived at work. It's a beautiful day, the sun is shining (not that you'll be seeing any of it
you hermit you 😉). You're raring to code. You switch on your laptop and grab a cup of coffee. You reply to a few emails
and see that you, *yes you*, have been hand-selected to review a pull request. Positively flattered, you open it up
only to be hit by a wall of red and green. You think, "what is this Lovecraftian-sized beast that stands before me?". A
changeset so big that you could develop RSI just by scrolling through it. Lucky for you, there are multiple requested
reviewers. So you close the tab and forget about it.

20 minutes later…

![Screenshot 1](/images/keep-those-prs-small/screenshot-1.png)

You've been sucked right back in. There's no escaping this now.

Headphones? Check. More coffee? Check. A copious number of biscuits? Check.

You open the PR again.

The summary stares back at you. Taunting you:

![Screenshot 2](/images/keep-those-prs-small/screenshot-2.png)

As you scroll through, you realise that despite what the description says, this PR doesn't just add a new
feature. There are refactorings, renamings, deleted files, reformattings, import reorderings, dependency upgrades, fixed
typos, dead code removal and, finally, new code. And not just new code. A *lot* of new code. New code that could have
been submitted independently in bite-sized chunks.

You spend the next 25 minutes going over the changes, trying to understand how all of this works whilst ducking and
weaving through superfluous updates. You gloss over certain parts (because life is too short). You leave a few
suggestions. There's a bit of back and forth between you and the author. The code generally looks pretty good. And
with that, you bestow the honour of a pull request approval. You're free at last. The monster is slain!
![Party Parrot](/images/keep-those-prs-small/parrot.gif)

1 hour later…

![Screenshot 3](/images/keep-those-prs-small/screenshot-3.png)

You take a look at this new PR. You are overcome with joy and relief when you see:

![Screenshot 4](/images/keep-those-prs-small/screenshot-4.png)

It's a fraction of the size of the previous PR. It's to the point. It's a bug fix and *only* a bug fix. Nothing more.
Nothing less. You see what the problem originally was and how this PR fixes it. You understand it perfectly. You see
how all the unit tests cover each edge case and will stop this bug from rearing its ugly head in the future. Reviewing
this PR is a doddle and you submit your approval in no time.

But you're left with an underlying feeling of disappointment. Disappointed that neither you nor the author spotted this
bug in the original PR. It was glaringly obvious (albeit surrounded by a bazillion other changes). Maybe it was just a
one-off.

30 minutes later…

![Screenshot 5](/images/keep-those-prs-small/screenshot-5.png)

…of course, it wasn't a one-off. There is rarely just one bug.

And so on and so on.

Now, this may be slightly exaggerated but I'm sure most of you have been in similar situations (having played both
characters). I'm using this as a way to illustrate why I believe that keeping your PRs small will lead to faster and
better quality reviews.

## So what is a "Small PR"?

For me, pull requests should do one thing. Let me paraphrase the advice in
[Clean Code](https://www.oreilly.com/library/view/clean-code/9780136083238/) on functions:

> Pull requests should do one thing. They should do it well. They should do it only.

Every line in your PR should contribute to that one thing. Anything that does not contribute to that one thing should be
put on a separate PR.

Mixing bug fixes and new features? **Put them on separate PRs.**

Moving those classes that we aren't using to a new package? **Put them on a separate PR.**

Those refactorings that have no relevance to this new feature? **Put them on a separate PR.**

Deleting old files that haven't been needed for the past 3 years? **Put it on a separate PR.**

Writing two services that are completely independent of each other? **Put them on separate PRs.**

Choosing to do one thing in your PRs will naturally make them more focused and smaller (and reviewers will adore you
for it!).

## The Good

So why on earth should we go to the effort of making smaller PRs?

### More people are likely to review

### More digestible == Quicker reviews

Smaller PRs are easier to digest. There's no bouncing around files, infinite scrolling or context switching. If a
reviewer understands your intention for the PR then they can offer more constructive criticism.

Not just more digestible for the reviewer but also for yourself. How many times have you created a big pull request,
read over it and thought, "what the hell am I even doing here?".

### Easier to spot problems

Time for a really basic example.

Give yourself a minute or so to read through the following code and hunt down the fairly obvious error:

```
public void initialiseCells() {
  for (int row = 0; row < grid.getRows(); row++) {
    for (int column = 0; column < grid.getColumns(); column++) {
      initialiseCell(new Coordinates(row, column));
    }
  }
}

private void initialiseCell(Coordinates coordinates) {
  Cell cell = new Cell();

  addAlivePropertyListener(cell);
  addClickEventHandler(cell);

  grid.add(cell, coordinates);
}

private void addAlivePropertyListener(Cell cell) {
  cell.aliveProperty()
    .addListener(newValue -> setAliveStyle(cell, newValue));
}

private void setAliveStyle(Cell cell, boolean isAlive) {
  List<String> styleClass = cell.getStyleClass();
  if (isAlive) {
    styleClass.add(ALIVE_STYLE_CLASS);
  } else {
    styleClass.remove(ALIVE_STYLE_CLASS);
  }
}

private void addClickEventHandler(Cell cell) {
  cell.addEventHandler(MouseEvent.MOUSE_CLICKED,
    event -> cell.toggleAlive());
}

public void resetCells() {
  for (int row = 0; row < grid.getRows(); row++) {
    for (int column = 0; column < grid.getRows(); column++) {
      resetCell(new Coordinates(row, column));
    }
  }
}
```

Did you find it? How about now:

```
public void resetCells() {
  for (int row = 0; row < grid.getRows(); row++) {
    for (int column = 0; column < grid.getRows(); column++) {
      resetCell(new Coordinates(row, column));
    }
  }
}
```

That second call to `grid.getRows()` should actually be `grid.getColumns()` (but you knew that, right?). How much
simpler was it to find the bug the second time around?

Admittedly, our unit tests should catch this. But what if the test data only uses square grids? Or even worse, what if
this code isn't even covered by our tests?

Bugs like this can go unnoticed as we focus on seemingly more complicated code when reviewing PRs. It's surprising how
often bugs creep into what should be trivial code. And this was in just 40(ish) lines. What happens when we get into the
world of reviewing complex classes that are 400 lines long? Or multiple complex classes that are 400 lines long? It's
easy to see how spotting bugs in a mammoth PR can be like trying to find
[hay in a needlestack](https://www.youtube.com/watch?v=WCheFdOy0tg).

*Small PRs make it easier for you as an author and as a reviewer to spot problems.*

### Constantly merging code into master

### A "truer" commit history

Git commits have a message for a reason.

// TODO: include image of git history and then an image showing different changes.

*Small PRs will help you to maintain a "truer" commit history.*

### Easier to roll back and pinpoint specific changes

// TODO: ties in nicely with last point. We can pinpoint specific changes.

### More encouraging for newbies (open source)

// TODO: contributions are valuable. No matter how big or how small.

## The Bad (maybe)

### No wider context

Link to related PRs. If you are calling a function in a different PR, how is this different from using built-in
libraries?

### Slow build process

This is a problem with your build process. Not small PRs.

### More noise?

Receiving more PRs, is this annoying?

## How?

Questions to ask yourself before submitting a PR.

> If I were asked to review this PR, would I be happy about it?

If the answer is 'No', then split into smaller PRs.

> Does this PR do one thing?

If the answer is 'No', then split into smaller PRs.

## Final thoughts

Be pragmatic. You don't have to create a separate PR for each individual function. As a rule of thumb, I would suggest
that if you're changing more than 10 files in a PR, then you should ask yourself, "am I doing too much?".

Give it a try for a week. Gauge the speed at which you are merging your PRs into master. Gauge how reactive people are
to reviewing your PRs. Gauge how often you're having to go back and fix bugs introduced by prior PRs. At the very least,
your number of contributions will start to look pretty healthy.

So one last time: **keep those PRs small!**

Thanks for reading.
