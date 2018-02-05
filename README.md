# Setup

## Part one: configuring nbstripout on a source repo

### Prerequisite: install `nbstripout`

To use nbstripout on a single repository, first make sure you have `nbstripout`
installed (`pip install nbstripout`) and make sure it's available in the
version of python you are using locally (`python -m nbstripout --help`).

### Set up the local git configuration

The following snippet will create a filter `ipynb_stripout` that uses
`nbstripout`, and configures git to use it for all `.ipynb` files:

```bash
cat >> .git/config <<EOF
[filter "ipynb_stripout"]
	clean = python -m nbstripout -t
	smudge = cat
	required = true
EOF

cat >> .gitattributes <<EOF
*.ipynb filter=ipynb_stripout
EOF
```

#### For the curious, what does the configuration above mean?

You can set up filters in `.git/config`, as we did.

The `clean` portion of the filter is an operation git performs on local
files prior to interacting with the git database. This means `clean` gets
run before a commit, but it also is run before other commands like `diff`.
Note that the local file is unchanged by `clean`; git simply uses the
output of `clean`, rather than the raw file contents, when interacting with
internals.

The `smudge` portion of the filter is what happens to a file stored in the
`git` database when you check it out; there are situations where you'd want to
run this, but for our purposes the files in `.git` are already clean, so we
just want to check them out, hence our use of `cat`.

The `required` flag indicates whether git should fail when commands do
not succeed (the most likely cause is the user not having installed a command,
e.g. if they have not installed `nbstripout` in our case).

Once a filter is set up, we can tell git to use it by creating an attribute
pairing in `.gitattributes`.

The git attribute does not require full `**/*` style globbing; the example
repo includes a notebook in a subdirectory to verify this.

### Assumptions about your system

Note: many guides to using nbstripout suggest using an absolute path to
the `nbstripout.py` file. I recommend against this because if you're using
ipython notebooks on a team of devs, it would likely be difficult to get
a hard-coded path to work for all of them. This is especially true if you
are working with a heterogeneous organization (I made this demo for
[Delta Analytics](deltanalytics.org)) rather than a company that can
standardize developer tools.

This guide also assumes that the command to invoke a python with a valid
`nbstripout` is simply `python`; depending on how you have your system and
paths set up you might need `python3`; I *highly* recommend using `pyenv` with
`pyenv-virtualenv`, which standardizes project setup so that a vanilla
`python` always works.
