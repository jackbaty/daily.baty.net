---
title: "Org-download"
lastmod: 2022-03-23T09:41:53-04:00
tags: ["emacs"]
draft: false
weight: 0
---

[abo-abo/org-download: Drag and drop images to Emacs org-mode](https://github.com/abo-abo/org-download)

**TL;DR** Just use 'attach mechanism and let files fall where they may


## Configuring org-download save directory <span class="tag"><span class="emacs">emacs</span></span> {#configuring-org-download-save-directory}

When I drag and drop an image into Emacs, I want the attached file to end up in `./img/YYYY/`. This is how I tried configuring it in my setup (I use Doom Emacs):

```emacs-lisp
  (setq org-download-method 'directory
        org-download-image-dir (concat "img/"  (format-time-string "%Y") "/")
        org-download-image-org-width 600
        org-download-heading-lvl 1)
```

For some reason, `org-download-method` was being reset from `'directory` to `'attach` after loading, and this broke things. I thought maybe I needed to set the variables _after_ org-download was loaded, so I did this:

```emacs-lisp
(after! org-download
  (setq org-download-method 'directory
        org-download-image-dir (concat "img/"  (format-time-string "%Y") "/")
        org-download-image-org-width 600
        org-download-heading-lvl 1))
```

That didn't work. At startup I was seeing this error:

> Error (org-mode-hook): Error running hook "org-fancy-priorities-mode" because: (void-variable org-download-image-dir)

Huh. I guess not everything can be set after org-download, so I tried only setting `org-download-method`

```emacs-lisp
(after! org-download
  (setq org-download-method 'directory))
```

This worked. Other settings are done in the `(after! org` block.

**Update March 22, 2022**: I guess it's only `org-download-image-dir` that causes problems. This seems to work fine&#x2026;

```emacs-lisp
(setq org-download-image-dir  "img/")
(after! org-download
   (setq org-download-image-org-width 800)
   (setq org-download-image-html-width 800)
   (setq org-download-heading-lvl 1)
   (setq org-download-timestamp "")
   (setq org-download-method 'directory))
```


## Issues with org-download-method and export {#issues-with-org-download-method-and-export}

After finally getting everything just the way I like it, I discovered that images aren't shown correctly when exporting to HTML or PDF. There's a [Github issue](https://github.com/abo-abo/org-download/issues/151) that looks related, but no response from the developer.

If I change `org-download-method` to 'attach' it's a whole set of other issues. I can't get file paths where I want them without setting the `DIR` property everywhere. But exports work.

```emacs-lisp
;; Org Download and Attachments
(setq org-download-image-dir  "img/")
(after! org-download
   (setq org-download-image-org-width 800)
   (setq org-download-image-html-width 800)
   (setq org-download-heading-lvl 1)
   (setq org-download-timestamp "%Y%m%d-")
   (setq org-download-method 'attach))

(setq org-attach-preferred-new-method 'dir)
(setq org-attach-id-dir  "files/")
(setq org-attach-dir-relative t)
;;   (setq org-attach-auto-tag nil)
(setq org-attach-store-link-p t)
(setq org-id-method 'ts)
;; for sane attachment paths/names
(setq org-attach-id-to-path-function-list
     '(org-attach-id-ts-folder-format
       org-attach-id-uuid-folder-format))
```