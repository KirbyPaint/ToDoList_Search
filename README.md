# _MVC Search Functionality_

## _Learn how to implement Search functionality in your MVC projects_

#### By _**Ash Porter (KirbyPaint)**_

## Description

_Learn how to implement a really cool feature in your MVC projects_

## Setup/Installation Requirements

I think you already know ;)

## How To Implement In Your Own Code

There are two key things that you will need to have to make a search bar function.  

1. A Search Bar  
2. Corresponding Controller route  
Now, this search bar can actually take a few forms, but in my project here, it is an actual search bar. Type in it, and magic happens.  
You can see this in my directory by navigating to Views/Items/Index.cshtml:  

```
// Index.cshtml View

@using (Html.BeginForm())
{
  <p>Search Items</p>
  @Html.TextBox("searchString")
  <input type="submit" value="Search" />
}
```

It's using a form to submit, of course, with a text box and a submit button. Not too much to explain here. The "Search Items" text can be anything you choose, and the button can have any `value` you want it to have. Just make sure to note whatever you name the TextBox itself, which in our case here, is called `searchString`.

Now, see the code from the Controller, in Controllers.ItemsController.cs:

```
// ItemsController.cs Controller

    public async Task<IActionResult> Index(string searchString)
    {
      var items = from model in _db.Items
                  select model;

      if (!(String.IsNullOrEmpty(searchString)))
      {
        items = items.Where(model => model.Description.Contains(searchString));
      }
      return View(await items.ToListAsync());
    }
```

Okay, time to explain a few things. Most importantly, and you will notice this if you go to the Controller page, this Index route is effectively replacing the original Index route. Notice how the parameter `string searchString` is named IDENTICALLY to the text box `@Html.TextBox("searchString")` in the View. This is _absolutely necessary_ or else none of this will work. Call it what you want, but call it the same thing each time.  

Now, there's a little bit of explaining the next few lines. `var items` is just querying your database, so this one is set to query everything in the database, and you can realistically do anything you want with that.  

This next bit is a little tricky, but all you really need to do is implement it in your page, play around a little bit, and understanding will come with time.

```
if (!(String.IsNullOrEmpty(searchString)))
      {
        items = items.Where(model => model.Description.Contains(searchString));
      }
```

In layman's terms, this section is basically searching the database for your search query.

`if (!(String.IsNullOrEmpty(searchString)))`
This is a conditional, using a built-in String function, to basically check if the string is `null` or blank. It uses `!` to then flip the result, and now we have a line stating "If this `searchString` is `NOT` `null` or `empty`, then proceed." Thankfully, this little function has a very clearly-defined name.

`items = items.Where(model => model.Description.Contains(searchString));`
Now, this is a bit of fun stuff. This line basically asks "hey database, show me anything where the description contains my search query." Important things to note:  
`items` in this case is previously defined by you.  
`.Where()` is a Linq method, where you can use it as a conditional. It'll only show things in the database that match your query.  
`model` is simply part of an anonymous function. Call it anything you want, just again, be sure to be consistent with it.  
`model.Description` is SPECIFIC to my project here - the database that this project is pulling from SPECIFICALLY has a Description column. You will want to change this to match your project and your particular lookup query.  
`.Contains()` is another Linq query. This, in tandem with `.Where()` will actually perform the lookup. `.Contains()` looks for a value, and in this case, we are passing the value `searchString` from our textbox.

So, once again, this controller takes in a `searchString`, builds a query from the chosen database, and then asks the database for only the data `.Where()` the database column `Description` `.Contains()` our aforementioned `searchString`

## Additional Magic

Was a search bar not good enough? You want Action Links that will sort by an attribute, maybe you want to show only items that have a certain attribute, or you're just looking for a quick and easy way to alphabetize your data. Never fear - the solution is here:

```
// Add this to your View

@Html.ActionLink("Sort By Rating", "Index", new {searchString = "Rating"})
```

There are a few parts to this ActionLink, and if you check this repo, you will actually notice this isn't a part of this exact project, which is okay, people move on and work on different things, just a part of life.  
The important things to know here are:  
"Sort by Rating" - this is just the link text the user will see. This can be anything you want, but I recommend making it mean something substantial. Don't say "Sort by Rating" if this isn't what your link does.  
"Index" - this will route back to the Index page, in this same controller directory. If this link is implemented in the Items directory, it's going to pass the next parameter to the Items controller, to the Index route. Plan accordingly.  
`new {searchString = "Rating"}` - so here's some magic. Remember how we had the `searchString` from earlier? Well, this will act in functionally the same manner. When the user clicks this link, it's going to pass the searchString "Rating" into our search bar. But, there's no parameter in the controller to check for the term "Rating," the controller only functions based on the input string. If you were just to dump this in, the controller would, when clicked, display any item where the `Description` contains "Rating."  

SOLUTION: conditionals.

```
// Add this to your Controller

public async Task<ActionResult> Index(string searchString)
    {
      var userId = this.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
      var currentUser = await _userManager.FindByIdAsync(userId);
      if (searchString == "Rating")
      {
        var userRecipes = _db.Recipes.Where(entry => entry.User.Id == currentUser.Id).OrderByDescending(model => model.Rating).ToList();
        ModelState.Clear();
        return View(userRecipes);
      }
      else
      {
        if (!(String.IsNullOrEmpty(searchString)))
        {
          var userRecipes = _db.Recipes.Where(entry => entry.User.Id == currentUser.Id).Where(model => model.Ingredients.Contains(searchString) || model.Name.Contains(searchString)).ToList();
          return View(userRecipes);
        }
        else
        {
          var userRecipes = _db.Recipes.Where(entry => entry.User.Id == currentUser.Id).ToList();
          return View(userRecipes);
        }
      }
    }
```

Well, that's a lot...

But! Not incomprehensible.

First of all, if there are any issues, you might be missing some `using` statements. I'll list all of mine at the bottom, but if you're REALLY confused, go check out this repo's controllers and respective views:

https://github.com/KirbyPaint/RecipeBox.Solution

Okay. You'll notice that there's some funky stuff going on with the first two lines:
```
var userId = this.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
var currentUser = await _userManager.FindByIdAsync(userId);
```

This is purely for user authentication, and if your project does NOT incorporate authentication, then you will not need these lines.

```
if (searchString == "Rating")
```

Here we go. Now we've got some conditionals. Use a conditional for whatever search filter you want, and just make sure that you spell them EXACTLY the same. Same capitalization and everything, or the search filter doesn't work.

```
{
  var userRecipes = _db.Recipes.Where(entry => entry.User.Id == currentUser.Id).OrderByDescending(model => model.Rating).ToList();
  ModelState.Clear();
  return View(userRecipes);
}
```

More database sorting magic. The first line, that extremely long line, is basically doing two things:
Pull up the Database (this database was a recipe book specifically) `.Where()` the entry was created by the logged-in user (again, this is authentication - omit this if you're not authenticating) and then `.OrderByDescending()` by the `Rating` column, then use `.ToList()` to... list.

`.OrderBy()` is another Linq method, which will sort your selected items alphabetically or numerically. In my case, with the Recipe book, the `Rating` column was actually saved numerically (1, 2, 3, 4, 5), using an enumerator field to render it on-screen as (one, two, three, four, five). Using a standard `.OrderBy()` query would sort the list with Recipes with a 1 rating at the top, and 5 at the bottom. However, most ratings tend to put the best-rated items at the top, so if we use `.OrderByDescending()` then we can just flip the order. This is great for A-Z sort.  

`ModelState.Clear();` is a handy little bit of code that is used to prevent the search bar's text box from getting confused. Since it and the ActionLink both are named "searchString", once the link is actually clicked and the sort is returned, the search bar will take on the text "Rating" or whatever you pass as the search parameter. This `ModelState.Clear();` just returns the textbox to a blank state.

Then, we get into the `else` of that first if-else block.

```
if (!(String.IsNullOrEmpty(searchString)))
{
  var userRecipes = _db.Recipes.Where(entry => entry.User.Id == currentUser.Id).Where(model => model.Ingredients.Contains(searchString) || model.Name.Contains(searchString)).ToList();
  return View(userRecipes);
}
```

Look familiar? It should. Functionally, this is the same search box as we had before, but with a twist. Check out the `.Where()` statement in that list:

```
.Where(model => model.Ingredients.Contains(searchString) || model.Name.Contains(searchString))
```

What we did here was to implement a conditional to our search box. Now, our `searchString` will be passed through both the Ingredients table AND the Name table. Basically, you can use OR ( || ) to search multiple tables in one go. Pretty neat!  

```
else
{
  var userRecipes = _db.Recipes.Where(entry => entry.User.Id == currentUser.Id).ToList();
  return View(userRecipes);
}
```

Finally, we have a remaining `else`, and basically this is our catch-all. If the search result didn't match any parameters, if the search was blank, if something weird happens, you should just end up with a standard listing of all your items.

## Known Bugs

If you were to have an ActionLink with a search parameter of "Rating" and you type "Rating" into the search bar and search with that, then it isn't going to search for items containing "Rating", it will actually perform the Rating sort as if you'd clicked the link. Keep that in mind.

## Support and contact details

_Discord: @KirbyPaint#0751_

### License Information

_GNU Public License_

Copyright (c) 2021 **_KirbyPaint_**  
Please feel free to contribute to this little project. Knowledge is meant to be shared with the world.