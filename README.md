# r-intermediate-penguins-dt

This is the repository for the [Carpentries demo lesson](https://carpentries.github.io/sandpaper-docs/introduction.html), modified to demonstrate usage of the [dovetail R package](https://github.com/joelnitta/dovetail) for translating Carpentries lessons.

## About [dovetail](https://github.com/joelnitta/dovetail)

`dovetail` is not yet on CRAN, and needs to be installed from GitHub:

```r
devtools::install_github("joelnitta/dovetail")
```

Also, note that `dovetail` relies on external software ([mdpo](https://github.com/mondeja/mdpo)) bundled in a [Docker image](https://hub.docker.com/r/joelnitta/mdpo), 
and therefore requires Docker to be installed and running.

## Translating a lesson

The below code demonstrates how you can generate the files needed for translation:

```r
library(dovetail)

# Copy (untranslated) files needed for rendering lesson to locale/{lang}/
# This will overwrite any translated files currently there.
create_locale("ja")

# Create PO files ----
# Either one file at a time; e.g., episode file
# md2po("episodes/introduction.Rmd", "po/ja/introduction.po")

# Or (recommended), create all PO files at once in `./po/{lang}/`
create_po_for_locale("ja")

# Edit PO files ----
# for example, with
# usethis::edit_file("po/ja/introduction.po")
# usethis::edit_file("po/ja/index.po")
# etc...

# Translate md files ----
# Either one at a time:
# po2md(
#   md_in = "episodes/introduction.Rmd",
#   po = "po/ja/introduction.po",
#   md_out = "locale/ja/episodes/introduction.Rmd")

# Or (recommended) translate all (R)md files at once to `./locale/{lang}/`
translate_md_for_locale("ja")

# Note that dovetail will still "translate" markdown files without any
# msgtr strings in the PO file. In other words, untranslated strings
# will show up in the output markdown file in their original language.

# Build translated lesson ----
sandpaper::build_lesson("locale/ja/")
```

If you run the above code without adding translations to the PO files, the 
"translated" lesson will appear exactly like the original lesson (in English).

I have partially translated lessons for Japanese.

If you run `translate_md_for_locale("ja")` followed by `sandpaper::build_lesson("locale/ja/")`, it will build the (partially) translated
lesson in `locale/ja/site`.

## Updating a translation

If you modify a markdown file of the original language, then run `md2po()` or `create_po_for_locale()`, only the corresponding parts that need updating in the PO file will change.
These will be marked with the comment `#, fuzzy`.
Once the translation has been updated, the `#, fuzzy` comment can be removed.

## Building website on GitHub

GitHub can be used to build the website with a workflow script.

Here is how to set it up to build a translation from a translation branch using R commands (do this after making the translation):

```
library(gert)

# Create ja branch
git_branch_create("ja")
git_branch_checkout("ja")

# Move translated directories to main dir
trans_dirs <- fs::dir_ls("locale/ja", type = "directory")
new_dirs <- fs::path_split(trans_dirs) |> purrr::map_chr(dplyr::last)
fs::dir_copy(trans_dirs, new_dirs, overwrite = TRUE)
fs::dir_delete(trans_dirs)

# Move translated files to main dir
trans_files <- fs::dir_ls("locale/ja", type = "file")
new_files <- fs::path_split(trans_files) |> purrr::map_chr(dplyr::last)
fs::file_copy(trans_files, new_files, overwrite = TRUE)
fs::dir_delete(trans_files)

# Commit changes and push
git_add(files = "*")
git_commit("Translate website")
git_push()
```

Note that you will also need to modify `.github/workflows/sandpaper-main.yaml` as follows:

```
on:
  push:
    branches:
      - main
      - master
```

to

```
on:
  push:
    branches:
      - ja
```

## File hierarchy

`dovetail` starts with the standard [sandpaper](https://carpentries.github.io/sandpaper/index.html)
file hierarchy, and modifies it as follows (indicated at bottom with `NEW`):

```
|-- .gitignore                  # - Ignore everything in the site/ folder
|-- .github/                    # - Scripts used for continuous integration
|   `-- workflows/              # - CI workflows
|-- episodes/                   # - Put your markdown files in this folder
|   |-- data/                   # -   Data for your lesson goes here
|   |-- fig/                    # -   All static figures and diagrams are here
|   |-- files/                  # -   Additional files (e.g. handouts) 
|   `-- introduction.Rmd        # -   Lesson markdown file
|-- instructors/                # - Information for Instructors
|-- learners/                   # - Information for Learners
|   `-- setup.md                # -   setup instructions (REQUIRED)
|-- profiles/                   # - Learner and/or Instructor Profiles
|-- site/                       # - This folder is where the rendered markdown files and static site will live
|   `-- README.md               # -   placeholder
|-- config.yaml                 # - Use this to configure commonly used variables
|-- CONTRIBUTING.md             # - Carpentries Rules for Contributions (REQUIRED)
|-- CODE_OF_CONDUCT.md          # - Carpentries Code of Conduct (REQUIRED)
|-- LICENSE.md                  # - Carpentries Licenses (REQUIRED)
|-- README.md                   # - Introduces folks how to use this lesson and where they can find more information.
|-- po                          # - NEW, contains PO files for translation
|   `-- ja/                     # - NEW, each subfolder is named by the target language (e.g., 'ja' for Japanese)
|       |-- introduction.po     # - NEW, each md file to be translated gets a corresponding PO file
|       |-- CONTRIBUTING.po     # - NEW, more PO files...
|       |...                    # - NEW, more PO files...
|-- locale                      # - NEW, contains translated files
|   `-- ja/                     # - NEW, each subfolder is named by the target language (e.g., 'ja' for Japanese)
|       |-- CONTRIBUTING.md     # - NEW, dovetail will generate the translated markdown files here
|       |-- site/               # - NEW, the translated, rendered site gets generated here
|           |-- built/          
|           |...               
```