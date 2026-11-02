# Rebuild and publish the 1968 exhibition with Git and GitHub Pages

Prepared November 1, 2026. Commands below use **Windows PowerShell**.

## 1. What this guide reproduces

**Exhibition:** 1968: A Nation at the Breaking Point  
**Live website:** https://gabby-colsia.github.io/1968-exhibition/  
**Repository:** https://github.com/Gabby-Colsia/1968-exhibition  
**Original published commit:** `832d97b82990d7f3b78f88e8ad99b15163f8c986`  
**Publishing settings:** public repository, `main` branch, `/ (root)` folder, GitHub Pages deployment from a branch.

There are two separate jobs:

1. **Reproduce the original:** use the exact HTML, images, and PDF supplied in this kit or stored at the original commit. This is checkable byte for byte.
2. **Make a revised version:** edit the writing or design, review it, commit it, and publish again. A deliberately revised file will no longer match the original fingerprint.

The text and images alone do not specify the exact layout, typography, colors, captions, navigation, or citation links. For exact reproduction, keep `index.html` too. Generating another site from the same material with an AI prompt is not a deterministic rebuild.

The original repository was created through GitHub's browser interface, and the completed files were uploaded and committed there. This guide also provides an equivalent command-line Git workflow; it is not a claim that the initial publication used those commands.

## 2. Understand the tools

| Tool or file | Its role |
| --- | --- |
| HTML | Defines the page's content and structure. |
| CSS | Defines appearance and responsive layouts. It is inside `index.html`. |
| JavaScript | Supplies the reading-progress indicator and print button behavior. It is inside `index.html`. |
| Git | Records versions of files on a computer. It does not write the history text or design the page. |
| GitHub | Stores the Git repository online. |
| GitHub Pages | Publishes files from the repository as a website. |
| `index.html` | The entry point that a browser opens for the exhibition. |
| SHA-256 fingerprint | Identifies a file's exact bytes so copies can be compared. |

This is a **static website**. There is no npm install, Node.js build, Python generation step, database, paid hosting, custom domain, or Codex dependency required to reproduce and publish the finished site. GitHub runs its own Pages deployment after the files are pushed.

## 3. Keep this complete rebuild kit

Extract `1968-rebuild-kit.zip` before starting. Use **Extract All**, rather than opening files inside the ZIP.

The extracted folder contains:

```text
1968-rebuild-kit/
    REBUILD-AND-PUBLISH.md       This guide
    SHA256SUMS.json             Expected hashes and sizes of the 17 original files
    Verify-Website.ps1          Checks a folder against those hashes
    site/
        index.html
        1968-exhibition-backup.pdf
        [15 JPEG images listed below]
    reference/
        content.json            Editable panel text and course-source locators
        visuals.json            Captions, creators, dates, source URLs and image metadata
```

`reference/` is an editing aid. The published HTML does not read those JSON files. Changing a JSON file does not update the website. The `site/` folder contains everything necessary for the existing website to work.

The private textbook PDFs are not included in this kit. Keep the assigned reading separately for checking the interpretation.

### Exact files in the website root

| File | Used for |
| --- | --- |
| `index.html` | Complete website, styling, navigation, panels and footnotes |
| `1968-exhibition-backup.pdf` | Downloadable backup |
| `2011661230.jpg` | Introduction: television coverage of Vietnam |
| `541874.jpg` | Tet: smoke in Saigon |
| `558530.jpg` | Tet: destruction in Cholon |
| `10694895.jpg` | Johnson's March 31 address |
| `2004665586.jpg` | McCarthy Peace poster |
| `186925188.jpg` | King in Memphis |
| `2003688169.jpg` | Poor People's Campaign marchers |
| `70740735.jpg` | Kennedy campaign handout |
| `69950431.jpg` | Kennedy portrait handout |
| `2016652537.jpg` | Chicago: demonstrator and National Guard |
| `2012647950.jpg` | Chicago: Illinois delegates |
| `2014645541.jpg` | Chicago: Senator Albert Gore Sr. |
| `2012647861.jpg` | Nixon campaign poster |
| `148146457.jpg` | Humphrey in New York |
| `192617.jpg` | Conclusion: Johnson listening to a recording |

All 17 belong in the **same directory** in this GitHub edition. Do not create an `assets/` folder or add a slash before an image filename. For example, `src="541874.jpg"` works under the project's URL; `src="/541874.jpg"` points to a different location.

## 4. Install and check Git

1. Install Git for Windows from [the official Git download page](https://git-scm.com/downloads/win) if it is not already installed. The standard installer includes Git Credential Manager for browser-based GitHub authentication.
2. Close and reopen PowerShell after installation.
3. Run:

```powershell
git --version
```

Expected result: a Git version number. If PowerShell says Git is not recognized, reopen the terminal or repair the installation before continuing.

Sign into the intended GitHub account in the browser. For the existing exhibition, the owner is **Gabby-Colsia**. Do not put a password or access token into any command in this guide.

## 5. First verify the saved source files

In File Explorer, open the extracted `1968-rebuild-kit` folder. Click the address bar, type `powershell`, and press Enter. PowerShell should open in that folder.

Run the following directly in PowerShell. Pasting these commands does not require enabling script files or changing the computer's execution policy:

```powershell
Get-Location
Get-ChildItem .\site
$baselineManifest = (Resolve-Path .\SHA256SUMS.json).Path
function Test-ExhibitionCopy {
    param([string]$SitePath)
    $expectedFiles = Get-Content -LiteralPath $baselineManifest -Raw -Encoding UTF8 | ConvertFrom-Json
    foreach ($entry in $expectedFiles) {
        $sourceFile = Join-Path $SitePath $entry.file
        if (-not (Test-Path -LiteralPath $sourceFile -PathType Leaf)) {
            throw "Missing file: $($entry.file)"
        }
        $actualHash = (Get-FileHash -LiteralPath $sourceFile -Algorithm SHA256).Hash.ToLowerInvariant()
        if ($actualHash -ne $entry.sha256) {
            throw "Changed file: $($entry.file)"
        }
    }
    'PASS: All 17 original exhibition files match byte for byte.'
}
Test-ExhibitionCopy -SitePath .\site
```

Expected result:

```text
PASS: All 17 original exhibition files match byte for byte.
```

Keep this PowerShell window open for the remaining commands; the function and the absolute manifest path remain available even when you change directories. If you reopen PowerShell, return to the kit directory and run this section again.

The check reports missing or changed files and fails if any baseline file differs. It allows extra files, so a later README will not invalidate the baseline. The companion `Verify-Website.ps1` provides the same check as a standalone script on computers that allow local scripts. It is optional; this guide uses the pasted function throughout.

## 6. Choose the right starting path

**Choose Path A** to restore, study, or edit Gabby's existing repository.  
**Choose Path B** to publish the identical website in a completely new repository or another person's account.  
Do not perform both paths in the same working folder.

### Path A: clone the existing repository

From the rebuild-kit directory, run:

```powershell
git -c core.autocrlf=false clone https://github.com/Gabby-Colsia/1968-exhibition.git exhibition-working-copy
Set-Location .\exhibition-working-copy
git config core.autocrlf false
git status
git remote -v
```

`clone` downloads the files and version history. Disabling automatic newline conversion preserves the original HTML bytes for the fingerprint check. The remote should point to Gabby's exhibition repository. A clean status means no local changes.

To inspect the **exact original** version even if the live site has changed since this guide was written, run:

```powershell
git switch --detach 832d97b82990d7f3b78f88e8ad99b15163f8c986
Test-ExhibitionCopy -SitePath .
```

Detached mode is for inspecting a historical snapshot. It does not change the published site. After reviewing it, return to the current editing branch before making new edits:

```powershell
git switch main
git pull --ff-only
```

Do not pull over unsaved edits. If Git reports a conflict or a diverged branch, stop and inspect `git status` before proceeding.

**If the assignment is simply to reconstruct the original site locally, Path A and the successful fingerprint check finish that step.** To edit the current site, continue to section 8. To recreate publication in a fresh account, use Path B instead.

### Path B: create a new repository and populate it from scratch

1. In the intended GitHub account, select **New repository**.
2. Set the owner to that account.
3. Enter `1968-exhibition-rebuild` as the repository name. This avoids colliding with Gabby's existing `1968-exhibition` repository. Another person may instead use `1968-exhibition` if that name is available.
4. Set visibility to **Public**.
5. Leave README, .gitignore, and license initialization off so the remote starts empty.
6. Select **Create repository**.

The following commands start from the extracted rebuild-kit directory. Replace `YOUR-USERNAME` with the actual repository owner's username and set the repository name to the one just created. Replace the sample commit email with an address verified on that GitHub account, or its exact GitHub-provided no-reply address from **Settings > Emails**. Git commit metadata is public in this public repository.

```powershell
$githubOwner = 'YOUR-USERNAME'
$githubRepository = '1968-exhibition-rebuild'
$commitName = 'YOUR NAME'
$commitEmail = 'YOUR VERIFIED OR GITHUB NO-REPLY EMAIL'

New-Item -ItemType Directory -Path .\exhibition-new-copy
Copy-Item -Path .\site\* -Destination .\exhibition-new-copy
Set-Location .\exhibition-new-copy

git init -b main
git config core.autocrlf false
git config user.name $commitName
git config user.email $commitEmail

Test-ExhibitionCopy -SitePath .
git add -- *.html *.jpg *.pdf
git status
git diff --cached --stat
git commit -m "Publish 1968 history exhibition"
git remote add origin "https://github.com/$githubOwner/$githubRepository.git"
git push -u origin main
```

Run each command in order and stop if a command reports an error. The directory must be new; if it already exists, choose a different working-folder name rather than overwriting another project.

The first authenticated push may open a browser sign-in through Git Credential Manager. Sign in as the owner of the new repository. A GitHub account password is not used as the HTTPS Git password. If authentication fails, use the credential manager's sign-in flow; do not paste secrets into the repository or this guide.

Open the new repository on GitHub. Confirm that `index.html`, all fifteen `.jpg` files, and the PDF appear in the file list. The new commit ID will differ from Gabby's original commit because its author, time, and repository history differ; the **file hashes** can still match exactly.

## 7. Preview the website locally

In the working copy, run:

```powershell
Start-Process .\index.html
```

This opens the static page in the default browser. No local server is required for this site's current features. Check the text, layout, navigation and images. Browser handling of the PDF's download button may differ for local files; verify that button again on the hosted HTTPS website.

For an unchanged rebuild, all 17 baseline files should still pass `Test-ExhibitionCopy -SitePath .` after previewing. Viewing a file does not change its contents.

## 8. Understand and edit the source

The publishable source is `index.html`. Open it in a plain-text code editor. Do not use Word to edit HTML.

Inside that file:

- `<head>` contains the browser title, description and CSS inside `<style>`.
- `<nav>` contains links such as `href="#tet"`.
- Sections have matching IDs: `introduction`, `tet`, `johnson`, `king`, `kennedy`, `chicago`, `election`, `conclusion`, and `sources`.
- The historical writing appears in ordinary paragraph tags, `<p>...</p>`.
- Each `<figure>` contains an image and caption.
- `<sup>` links connect numbered references to footnotes such as `id="fn-3"`.
- The final script updates the reading-progress bar. The print button calls the browser's print function.

To revise a sentence:

1. Search for the exact sentence in `index.html`.
2. Replace only the intended prose. Preserve surrounding HTML tags, links, quotation markup and footnote anchors.
3. Save as UTF-8 with the filename exactly `index.html`.
4. Reopen or refresh the local preview.
5. Check the matching source passage and citation.
6. Update the PDF backup to contain the same revised writing before submitting.

The `reference/content.json` file is useful for reading the original panel text without HTML. It is not a generator input for this published copy. If you use it to draft revisions, transfer approved revisions into the HTML too.

### Writing and source requirements to preserve

- Introduction: 150–250 words; original is 200.
- Six interpretive panels: 300–400 words each; the original panels are within that range.
- Conclusion: 150–250 words; original is 198.
- Fifteen embedded visual sources, each with a caption and numbered source note.
- A clearly stated Big Idea and a title.
- Written historical content based on Chafe's assigned chapter and class material.
- Chicago-style numbered notes, with creator, title or description, date, holding institution or reproduction site, and URL for visuals.

The course book is William H. Chafe, *The Unfinished Journey: America Since World War II*, ninth edition (Oxford University Press, 2022), chapter 12. The supplied chapter extract lacks printed book pagination. Existing notes explicitly distinguish positions in that 30-page PDF from printed book page numbers.

The assignment calls for a historical argument and interpretation as well as reporting events. Proofreading should therefore check both factual accuracy and whether the student agrees with the connections the exhibition makes.

### Keep the PDF synchronized

For an identical rebuild, copy the supplied PDF unchanged. It is part of the fingerprinted baseline.

After a text revision, one straightforward way to make a matching backup is to open the revised website, select **Print exhibition**, choose **Save as PDF**, and review every page. Save the reviewed result as `1968-exhibition-backup.pdf` in the website root, replacing only your working copy's backup. The browser-printed layout will differ from the original 16-page typeset PDF; it can still contain the revised written material required as a backup. Keep the original in the kit if you need to reproduce the baseline again.

Do not claim an edited or browser-regenerated PDF is byte-identical to the original. The original PDF is retained precisely so an exact rebuild does not depend on fonts, a PDF-generation library, or browser print settings.

## 9. Commit and push an approved revision

Use this section in a working copy on `main`, after reviewing the intended changes:

```powershell
git status
git diff -- index.html
git add -- index.html 1968-exhibition-backup.pdf
git diff --cached --stat
git commit -m "Revise exhibition after proofreading"
git push
```

If you intentionally changed an image, explicitly add that image filename too. If Git says there is nothing to commit, no tracked file has changed.

The review cycle is: **edit > preview > review sources > update backup > inspect diff > commit > push > verify deployment**. Do not force-push to publish ordinary edits.

## 10. Enable GitHub Pages

For an entirely new repository:

1. Open the repository on GitHub.
2. Select **Settings**.
3. In the sidebar, select **Pages**.
4. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
5. Select the **main** branch.
6. Select **/ (root)** as the folder.
7. Select **Save**.
8. Leave the custom-domain field empty. No purchased domain is needed.
9. Open the repository's **Actions** tab.
10. Open the **pages build and deployment** workflow and wait for **Success**. Its summary includes the live site link.

For Gabby's existing repository these settings are already configured. A normal push to `main` triggers a new deployment; you do not need to enable Pages again.

The standard project address is:

```text
https://YOUR-USERNAME.github.io/YOUR-REPOSITORY/
```

For example, the original is:

```text
https://gabby-colsia.github.io/1968-exhibition/
```

Use the **website** URL for Canvas, rather than the `github.com/...` repository URL. GitHub provisioning, interface labels and deployment duration can change; a green successful deployment and a working live page are the completion checks. See [GitHub's Pages instructions](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site).

## 11. Verify the published result

Before calling the job complete:

1. Open the published URL in a signed-out or private browser window.
2. Confirm that the title and Big Idea display.
3. Follow the navigation to all six exhibits.
4. Confirm that all fifteen images display and open when clicked.
5. Follow a numbered footnote and its return link.
6. Check several archival source links.
7. Download the PDF and open it.
8. Check the page on a narrow screen as well as a desktop screen.
9. Confirm the PDF contains the same written material as the website.
10. For an unchanged local rebuild, run the fingerprint check once more.

For an optional on-server byte check, start PowerShell in the rebuild-kit directory. Replace the sample site address with the deployment you are testing:

```powershell
$publishedBase = 'https://gabby-colsia.github.io/1968-exhibition'
$expectedFiles = Get-Content .\SHA256SUMS.json -Raw -Encoding UTF8 | ConvertFrom-Json
$downloadFolder = Join-Path (Get-Location) ('published-check-' + [guid]::NewGuid().ToString('N'))
New-Item -ItemType Directory -Path $downloadFolder | Out-Null
foreach ($entry in $expectedFiles) {
    $downloadPath = Join-Path $downloadFolder $entry.file
    Invoke-WebRequest -Uri ($publishedBase.TrimEnd('/') + '/' + $entry.file) -OutFile $downloadPath -UseBasicParsing
    $downloadedHash = (Get-FileHash -LiteralPath $downloadPath -Algorithm SHA256).Hash.ToLowerInvariant()
    if ($downloadedHash -ne $entry.sha256) {
        throw "Published file differs from baseline: $($entry.file)"
    }
}
'PASS: All 17 published files match the saved original.'
```

This creates a new verification folder without deleting anything. A network error is not proof that the file changed. A mismatch after deliberate edits is expected: the check compares with the original, not with a newly approved revision.

## 12. Troubleshooting

| Symptom | Check or next step |
| --- | --- |
| `git` is not recognized | Install Git and reopen PowerShell. |
| Push denied or wrong account | Check `git remote -v`; sign in through Git Credential Manager as the repository owner. |
| Remote already contains a README | Prefer cloning that repository and copying the site files into the clone. Do not force-push over its history. |
| Git asks for author identity | Set repository-local `user.name` and `user.email` as shown in Path B. |
| Site shows 404 | Wait for the build to finish; check Pages is `main` plus `/ (root)` and `index.html` is at the root. |
| Images are missing | Check exact filenames and capitalization. Keep filenames relative to the HTML. |
| Old text still displays | Confirm the new commit is on GitHub, its deployment succeeded, and then refresh the browser. |
| A check says the HTML changed after cloning | Check newline conversion settings. Exact-byte verification needs `core.autocrlf=false`; do not ignore the mismatch without understanding it. |
| PDF downloads old writing | Replace the working-copy PDF, commit it, push it and wait for deployment. |
| Build is red | Read the failing job in Actions. Do not assume publication succeeded because the push succeeded. |

See [GitHub's cloning guide](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) and [Pages 404 troubleshooting](https://docs.github.com/en/pages/getting-started-with-github-pages/troubleshooting-404-errors-for-github-pages-sites).

## 13. What “100% accurate reproduction” means here

The kit supports **identical files** and a repeatable publishing procedure. Identical file hashes prove that the source text, code, images, captions, citations and PDF bytes were preserved.

They do not guarantee identical pixels on every device: fonts, screen width, zoom and browser rendering can differ. They also do not guarantee GitHub uptime, deployment speed, continued availability of external archival links, or that a human historical interpretation is beyond revision. Those are separate checks.

Keep the ZIP, this guide and the baseline commit ID together. With those preserved, another person can recreate this version without access to the original Codex chat or its hosting account.
