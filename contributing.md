# Contribution Guidelines

Please note that this project is released with a [Contributor Code of Conduct](code-of-conduct.md).
By participating in this project you agree to abide by its terms.

-

Ensure your pull request adheres to the following guidelines:

- Search previous suggestions before making a new one, as yours may be a duplicate.
- Suggested _query_ or _command_ should be related to _MySQL database_ only.
- Make an individual pull request for each suggestion.
- Make sure that your _query_ or _command_ was tested before.
- Additions should be added to the bottom of the relevant category.
- Keep the title and description short and simple, but at the same time descriptive.
- Use the following _Markdown_ format:

  Query:
  
  <pre>
  ## Query Category
  ### Query Subcategory

  #### Query title...

  Query description... (optional)

  ```sql
  SELECT COUNT(`id`) FROM `foo` WHERE `foo`.`id` = 100000;
  ```

  _credits: [the author or source](URL)_ (optional)
  </pre>

  Command:

  <pre>
  ## Command Category
  ### Command Subcategory

  #### Command title...

  Command description... (optional)

  ```sh
  mysql -h 127.0.0 -u bar -p
  ```

  _credits: [the author or source](URL)_ (optional)
  </pre>

- The _credits: ..._ sould be in _italic_ with author or source url (optional).
- Don't mention _mysql_, _database_, _query_, _command_, _scripting_ or _terminal_ in the _title_ or _description_ as it's implied.
- Start the description with a capital and end with a full stop/period.
- The pull request should have a useful title and include the output or relevant screenshots of your command results and why it should be included.
- Check your spelling and grammar.
- Make sure your text editor is set to remove trailing whitespace.
- New categories, or improvements to the existing categorization or topics are welcome.

Thank you for your suggestions!

### Updating your PR

A lot of times, making a PR adhere to the standards above can be difficult. If the maintainers notice anything that we'd like changed, we'll ask you to edit your PR before we merge it. If you're not sure how to do that, [here is a guide](https://github.com/RichardLitt/docs/blob/master/amending-a-commit-guide.md) on the different ways you can update your PR so that we can merge it.
