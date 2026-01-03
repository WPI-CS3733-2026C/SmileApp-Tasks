# Warmup Project - CS 3733 - 2025 B Term

# Milestone2

---

### Task 1: Get Started

---

In Milestone2,  you will start with your milestone-1 code and add additional features.

1. First, make sure that your are checked out to `milestone1` branch ; run the following comment if you are not already.

      ```
      git checkout milestone1
      ```

   Create and checkout to a new branch called "milestone2".  The following commands will create a new branch "milestone2" and checkout to that branch. 

      ```
      git branch milestone2
      git checkout milestone2
      ```

   Make sure that your 'milestone2' branch now has the latest milestone1 code. You will now be editing locally, and **remember to commit frequently**.

2. In milestone-2, you will have to make changes to the following files:

   * `main.css`
   * `_post.html`
   * `base.html`
   * `index.html`
   * `create.html`
   * `forms.py`
   * `models.py`
   * `routes.py`

7. How to run the app:
   * First run the backend (this will run the app in debug mode):

     `python smile.py`

   * Open a browser (Chrome is recommended) and open the URL 'http://localhost:5000/'
     At this point, the page should look like the following.

   <kbd><img src="README.d/milestone2_task2a.png" width="800" border="2"></kbd>


---

### Task 2: Revise DB Model - Add `Tag` table

---

In our Smile Portal app, the users will be able to associate some predefined tags with their stories. We will store those tags in the `Tag` table and define a "many-to-many" relationship  between `Post` and `Tag`. The relationship will store the associations between post messages and the tags associated with them. Note that a Post can have multiple tags associated with it and a tag can be assoicated with many posts.

   1. In `models.py`, add a database model called `Tag`. The 'Tag' table will store the tags for the posts. It should have the following attributes:


      |   | Field name | Description | Field type | Constraints | Comments |
      | :- | :-: | :-: | :-: | :-: | :-: |
      |  | id | auto generated id | integer | primary key | id of the Tag |
      |  | name | tag name | string | max 20 chars | for example: "funny","inspiring", etc. |

   
   2. In `models.py`, add a SQLAlchemy association table called `postTags` (use `db.Table` SQLAlchemy funtion to create this table - for examples, see the `assigned` table in the [Flask class example](https://github.com/WPI-arslanay/FlaskLectureExample/tree/main/13-SQLAlchemy_ManytoMany) OR the `students_majors_table` table we defined in StudentApp application. )

      This table will store the associations between posts and the tags. Note that a post may be associated with multiple tags, and a tag can be associated to many posts. 

      <div style="border: 1px solid gray;  padding : 5px" >
         <i>Important note:</i> 
         In the file, `postTags` definition should be placed before both `Post` and `Tag` definitions. 
      </div>


      `postTags` should have the following attributes:

      |    | Field name | Description | Field type | Constraints | Comments |
      | :- | :-: | :-: | :-: | :-: | :-: |
      |    | post_id | id of the post | integer | foreign key to `post.id` |   |
      |    | tag_id | the id of the tag | integer | foreign key to `tag.id` |   |

      Also, add a `__repr__` method to `Tag` model for printing tags. You should print the id and name of each tag.  See the `__repr__` methods in StudentApp.

   3. Next, add a relationship attribute called `tags` to the `Post` model. `tags` will represent the collection of tags associated with a post. See the `Major.students_in_major` and `Student.majors_of_student` relationship declerations in StudentApp for examples.

      The `tags` relationship should be a `WriteOnlyMapped` attribute and it should reference to the `Tag` table through `postTags`. You should configure the other arguments of the  `tags` relationship as follows:

      ```python
      secondary = postTags
      primaryjoin=(postTags.c.post_id == id)
      ... 
      ```
   
      Similarly, add a relationship called `posts` to the `Tag` relationship. Make sure to assign the `back_populates` parameters both for `tags` and `posts`.   

   4. Define a method called `get_tags` in the `Post` class that returns **the list of tags** for a post. Remember that all methods of a class should have `self` as their first argument.

   5. Import `Tag` model and and `postTags`  table in `routes.py`.

   6. Next we will apply the changes in the schema to the database file. Run the following commands to migrate the changes and update the database.
   
   ```
      flask db migrate -m "Adding Tag table and many to many association between Tag and Post tables."
      flask db upgrade
   ```
   ---   
   <div style="border: 1px solid gray;  padding : 10px" >
   <b>Important note:</b>  
   
   *  If the above commands fail, or
   *  If you previously deleted the database file `smile.py` and did not initialize `flask migration` repository, 
      go ahead and delete the database file and have the application re-create it. 
   Remember that, in order to re-create the database with the new schema, you need to first delete the current database file `smile.db` and re-run the backend app. The decorator function `initDB` in `smile.py` will re-create the database file before the first request to the app.   

   After the database is re-created, no smile posts will be displayed on the main page.

   </div>
   
   ---   

   7. When the database is initialized, we would like to add some default tag names to the `Tag` table. 
      * Add a SQLAlchemy event listener for  the `Tag` table which will insert te below tag names to the database after it is created. See the `add_majors` function we defined in the StudentApp as an example. 

      ```Python
      tags = ['funny','inspiring', 'true-story', 'heartwarming', 'friendship']
      ```
      A copy of the above tag list is also available in the given skeleton code. The Flask app will run this function after it is created and will add the default tags to the database.
      * Make sure to import `Tag` table in `smile.py` file. 
      * **Delete the `smile.db` database file,**  re-run the backend app, and open main page on the browser (i.e., `http://localhost:5000/`). Make sure that `smile.db` file is re-created. 

  8. We next will insert some posts to the `Post` table on the Python command line.  We will create the posts acording to the modified `Post` model.

      * First, add the `Tag` model to the shell context so that it is imported in the flask shell environment. (i.e., add the following to the dictionary that `make_shell_context` function returns in `smile.py` file. ) 

         ```
         'Tag': Tag
         ```

      * Then open flask shell and run the following SQLAlchemy commands. These will query the `Tag` table and make sure that `Tag` is initialized with the default tags:
   
         ```python
         > flask shell
         >>> all_tags = db.session.scalars(sqla.select(Tag)).all()
         >>> for t in all_tags:
         >>>     print(t.name)
         ```

      * You should see the below tags printed.

         ```
         funny
         inspiring
         true-story
         heartwarming
         friendship
         ```
      * Now we will add some post data and associate them with tags.

         ```python
         # Create a new post
         >>> p = Post(title="Test post-1", body = "First test smile post. Don't forget to smile today!", likes=0, happiness_level = 3)
         # Write the new post to the database
         >>> db.session.add(p)
         # Get the `tag` object with `name` "funny" 
         >>> t1 = db.session.scalars(sqla.select(Tag).where(Tag.name == "funny")).first()
         # Add it to the `tags` relationship of the new post
         >>> p.tags.add(t1)
         >>> db.session.commit()
         # Get the `tag` object with `name` "heartwarming" 
         >>> t2 = db.session.scalars(sqla.select(Tag).where(Tag.name == "heartwarming")).first()
         # Append it to the `tags` relationship of the new post
         >>> p.tags.add(t2)
         >>> db.session.commit()
         >>> 
         ```

         Note that, we first query the tag table to get some tags, then we associate them with the posts by calling the `add` function for the `tags` relationship, i.e., `p.tags.add()`.

      * Now we will retrieve and print the tags of a particular post.

         ```python
         >>> p1 = db.session.scalars(sqla.select(Post).where(Post.title == "Test post-1")).first()

         >>> for t in p1.get_tags():
         >>>     print(t.name)
         ```
      * This should print the following.

         ```
         funny
         heartwarming
         ```

---

### Task 3: Associating tags with posts

---

In Task 3, we manually associated tags with posts. Now we will update the `PostForm` in forms.py and create.html template. We will allow users choose some tags for their post when they create a new story. We will also update  `_post.html` and display the tags associated with each post.

   1. In `forms.py`, add a new field named `tag` to `PostForm` (before `submit`). This should be a `QuerySelectMultipleField` as shown below. The tag declaration is not complete; you need to define the lambda functions for `query_factory` and `get_label` fields.

      * `query_factory` should be assigned to a function that returns all tags in the `Tag` table.

      * `get_label` should be assigned to a function that takes a `Tag` object as argument and returns the name for that tag.

         ```python 
            tag =  QuerySelectMultipleField( 'Tag', query_factory=......... , get_label=........, widget=ListWidget(prefix_label=False), 
               option_widget=CheckboxInput() )
         ```
         See [https://wtforms-sqlalchemy.readthedocs.io/en/stable/wtforms_sqlalchemy/](https://wtforms-sqlalchemy.readthedocs.io/en/stable/wtforms_sqlalchemy/) for more information on `QuerySelectMultipleField`.

      * Make sure to import the following in `forms.py`:
        * `QuerySelectMultipleField` from `wtforms_sqlalchemy.fields`
        * `ListWidget` and `CheckboxInput` from `wtforms.widgets`
   
      * Also, make sure to import model `Tag` in `forms.py`. 

   2. In `create.html` add the "tag" form element to the form template. It should be after "happiness_level" select box and before "submit button.
    
   3. In `routes.py` revise the `main.postsmile` route. Before the post is added to the database, add the selected tags to the `Post.tags` (similar to what we have done in Task 3 on the Python command).
   Note that, in the form instance, `form.tag.data` will hold the list of all selected tags (selected by user). You should iterate over each tag in `form.tag.data` and add them to post's `tags` relationship.

   4. When you open the "Post Smile" page on the browser (i.e., `http://localhost:5000/postsmile`), the page will look like the following:.

      <kbd> <img src="README.d/milestone2_task4.png" width="900" border="2"> </kbd>

   5. Run your app and **create a post by filling out the "create post form"**. Select multiple tags for your story.

      You can verify that tags are added correctly to your post by running the SQLAlchemy commands on the Python prompt, similar to what we did in Task 3. (In the below command, change the title in `where` clause to retrieve other posts.)

      ```python
      > flask shell 
      # change the title in the below query to the title the new story you created. 
      >>> p1 = db.session.scalars(sqla.select(Post).where(Post.title == 'The story title')).first()
      >>> for t in p1.get_tags():
      >>>     print(t.name)
      ```
      The `print` statement should print the tags you selected when you filled out the "Create Post" form. 
---

### Task 4: Displaying tags

---

We will now update  `_post.html` and display the tags associated with each post.

1. Include a Jinja2 `for` block in `_post.html` and display all the tag names associated with the post (Hint: use  `post.get_tags()`. This will give you the collection of tags. You need to just display the name of each tag. )

   The line that you should add your code is marked with the following comment.

   ```html
       <!-- (milestone2) TODO: include all the tags associated with the post-->
   ```
   When you open the main page on the browser (i.e., `http://localhost:5000/`), the page will be similar to the following:.

   <kbd> <img src="README.d/milestone2_task5a.png" width = "900" border="2"> </kbd>

2. Now we will style the tag names.

   Add a class selector `tagitem` in  `main.css` . We will use this to style the tag names in  `_post.html`.
   Add styling to adjust the text alignment,  margin, padding, border, and background-color.

3. In `_post.html`, enclose the tag name in a `<span>` element and style it with class selector `tagitem` .

   Create other posts and associate tags with your posts. 
   
   When you open the main page on the browser (i.e., `http://localhost:5000/`), the page will be similar to the following:.

   <kbd> <img src="README.d/milestone2_task5b.png" width="900" border="2"> </kbd>

---

### Task 5: Sorting posts

---

Lastly, we will add sorting functionality to our app. The user will be able to sort the posts by:

   * post timestamp (i.e., date)- in decending order (default sort order)
   * post title - in ascending order,
   * number of likes - in decending order, and
   * happiness level - in decending order.

1. In `forms.py`, add a new form class named `SortForm`. We will use `SortForm` to get the user's selection for the sort attribute.     

   `SortForm` should have a `SelectField` (with `choices`  'Date', 'Title', '# of likes', and 'Happiness level') and a `SubmitField` with label "Refresh".

2. In `index.html`, add template to render an instance of a `SortForm`. (You will create the form instance in step-3 and pass it to `render_template` function when `index.html` is rendered. ) 
   
   The form should display the sort attribute select-box and the "Refresh" submit button. 
   
3. In `routes.py`, edit the `main.index` route function. 
   
   * Add `'POST'` to the route `methods`. 
   * Create a `SortForm` instance in the  `index()` route function. 
   * When the form is submitted, you should retrive all posts from the database sorted based on the search field that was sent in the form. 
      * **Hint:** You should use the `order_by` function of the SQLAlchemy query object. Remember that, you need to pass a SQLAlchemy model column as argument to `order_by`. Include the `.desc()` after the column to sort in decsending order. [https://stackoverflow.com/questions/4186062/sqlalchemy-order-by-descending](https://stackoverflow.com/questions/4186062/sqlalchemy-order-by-descending)
   * Otherwise, all posts should be sorted by most recent timestamp. 
   * You can use helper functions to get the sort attribute. For example given the sort option, your helper function will return the corresponding `Post` field. 
   * Make sure to import `SortForm` in routes.py.
   
    ---  
    <div style="border: 1px solid gray;  padding : 10px" >
      <b>Remember:</b> 
      The `SortForm` instance you created needs to be passed to the `render_template` call for rendering. 
    </div>
   
   ---


   <div style="border: 1px solid gray;  padding : 10px" >
   <b>Important Note:</b>

   * The data recieved in a post request (e.g., `form.choice.data`) will always be a string. So, even though a particular field of the form has a numberic value, the recived data will be in string format.
   To understand the `form` data you receive in the POST response  and make sure to print this value in your route view function. If the `SelectField` in your sort form is supposed to return you an `integer` value, you will receive it as `string` and you may convert it to an `int` value by using the Python `int()` function.    
   </div>


   ---

4. Add styling to the form elements:
   * In `index.html` style the sortorder form element using the `formselect` CSS element we defined in milestone1. 

   
   See the below image for an example styling.

   <kbd> <img src="README.d/milestone2_task5c.png" width="900" border="2"> </kbd>

   ---
   <div style="border: 1px solid gray;  padding : 5px" >
      <b>Note:</b> 
      After sorting stories, if you "like" a story the page will be reloaded and the stories will be sorted by the default sort order (i.e., timestamp). In milestone3, we will refactor our "like" use case and update the like counts without reloading the page (using JavaScript).
    </div>
   
   ---


   Add couple additional smiles posts and make sure that the stories are sorted correctly.  When you complete milestone2 , the main page will be similar to the following:

   <kbd> <img src="README.d/milestone2_final.png" width="900" border="2"> </kbd>


---
### How to Submit
---
1. Commit and push to GitHub

   *  Make sure you are in branch `milestone2`, and check the commit status. 
       ```
       git checkout milestone2
       git status
       ```
   * Add and commit your changes locally. Make sure you are in branch `milestone2`. 
       ```
       git add <list all new and changed files here; seperate filenames with space.>   (or  git add .)
       git commit -am "Your own commit message"
       ```

   * Push the `milestone2` branch to your remote GitHub repo:
       ```
       git push origin milestone2
       ```
       - You can commit multiple times. Please make sure not to make any commits to `milestone2` branch after the milestone2 deadline. 
  

2. Submit your repo link on Canvas

   * Copy your repository URL and submit it in the Canvas `Milestone2` dropbox. 
 
