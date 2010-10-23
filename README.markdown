mk-project.el: An Emacs project library
=======================================

What is it?
-----------

[mk-project.el](http://www.emacswiki.org/cgi-bin/wiki/mk-project.el) is an Emacs library to quickly switch between projects and perform operations on a per-project basis. A project in this sense is a directory of related files, usually a directory of source files. Projects are configured in pure Emacs lisp and do not require configuration files in the project's directory.

The "mk" in mk-project used to stand for me, Matt Keller, but now that I've published this library, let's say it stands for "make" as in "make project".


How does it work?
-----------------

A project is configured in 100% pure elisp. For example, the following elisp code...

    (project-def "my-java-project"
          '((basedir          "/home/me/my-java-project/")
            (src-patterns     ("*.java" "*.jsp"))
            (ignore-patterns  ("*.class" "*.wsdl"))
            (tags-file        "/home/me/.my-java-project/TAGS")
            (file-list-cache  "/home/me/.my-java-project/files")
            (open-files-cache "/home/me/.my-java-project/open-files")
            (vcs              git)
            (compile-cmd      "ant")
            (ack-args         "--java")
            (startup-hook     my-java-project-startup)
            (shutdown-hook    nil)))
    
    (defun my-java-project-startup ()
      (setq c-basic-offset 3))

...defines a project called "my-java-project" under the /home/me/my-java-project directory with additional, optional settings to help identify which files should be included in your TAGS file, which files should be excluded from a grep operation, etc.

Once a project is configured via `project-def` and loaded via `project-load`, the following project functions can be used:

<table border="1">
<tr>
  <th>Function</th>
  <th>Recommended Key Binding</th>
  <th>Description</th>
</tr>
<tr><td>project-<b>load</b></td>
    <td>C-c p l</td>
    <td>Set project variables, open files named in the open-files-cache, run the user-defined startup hook.</td>
</tr>
<tr>
  <td>project-<b>unload</b></td>
  <td>C-c p u</td>

  <td>Save open files names to the <i>open-files-cache</i>, run shutdown hook, run the user-defined shutdown hook, 
    unset project variables, prompt to close all project files.</td>
</tr>
<tr>
  <td>project-<b>compile</b></td>
  <td>C-c p c</td>
  <td>Run the project's <i>compile-cmd</i> with the given arguments.</td>
</tr>
<tr>
  <td>project-<b>grep</b></td>
  <td>C-c p g</td>
  <td>Run <code>find-grep</code> on the project's <i>basedir</i>, ignoring any files that match
    <i>ignore-patterns</i> or are excluded by the <i>vcs</i> setting. mk-project.el knows how to avoid the
    VCS-specific files and directories of git, cvs, subversion, mercurial and bzr. If given a C-u argument,
    start the search from the current buffer's directory.</td>
</tr>
<tr>
  <td>project-<b>ack</b></td>
  <td>C-c p a</td>
  <td>Run <a href="http://betterthangrep.com">ack</a> from the project's <i>basedir</i>.
    If given a C-u argument, run ack from the current buffer's directory.</td>
</tr>
<tr>
  <td>project-<b>multi-occur<b></td>
  <td>C-c p o</td>
  <td>Search open project files using 
  <a href="http://www.gnu.org/software/emacs/manual/html_node/emacs/Other-Repeating-Search.html">multi-occur</a>.</td>
</tr>
<tr>
  <td>project-<b>find-file</b></td>
  <td>C-c p f</td>
  <td>Quickly open a file in the project's <i>basedir</i> by regex. The search
    excludes files matching <i>ignore-patterns</i> or excluded by the <i>vcs</i> setting.</td>
</tr>
<tr>
  <td>project-<b>find-file-ido</b></td>
  <td>C-c p f</td>
  <td>Quickly open a file in <i>basedir</i> using <a href="http://www.emacswiki.org/emacs/InteractivelyDoThings">ido</a>.</td>
</tr>
<tr>
  <td>project-<b>index</b></td>
  <td>C-c p i</td>
  <td>Re-index the project files. The index is stored in buffer *file-index*
    and is used by <code>project-find-file</code>. The index can be cached to the file set in <i>file-list-cache</i>.</td>
</tr>
<tr>
  <td>project-<b>tags</b></td>
  <td>C-c p t</td>
  <td>Regenerate the project's TAGS file. Only files matching <i>src-patterns</i> are
    indexed (see also <i>src-find-cmd</i>).</td>
</tr>
<tr>
  <td>project-<b>dired</b></td>
  <td>C-c p d </td>
  <td>Open dired on the project's <i>basedir</i>.</td>
</tr>
<tr>
  <td>project-<b>status</b></td>
  <td>C-c p s</td>
  <td>Print values of project variables</td>
</tr>
<tr>
  <td>project-<b>menu</b></td>
  <td></td>
  <td>Enable the 'mk-project' menu.</td>
</tr>
<tr>
  <td>project-<b>menu-remove</b></td>
  <td></td>
  <td>Disable the 'mk-project' menu.</td>
</tr>
</table>

mk-project.el relies heavily on the find and grep commands in your environment. Unix and Linux systems certainly have find and grep. Cygwin can provide these commands for Windows.

The following table describes the configuration directives that are used in `project-def`.

<table border="1">
<tr>
  <th>Directive</th>
  <th>Required/Optional</th>
  <th>Description</th>
</tr>
<tr>
  <td>basedir</td>
  <td>Required</td>
  <td>Base directory of the current project. Value is expanded with <code>expand-file-name</code>. Example: "/home/me/my-proj/"</td>
</tr>
<tr>
  <td>src-patterns</td>
  <td>Optional</td>
  <td>List of shell patterns to include in the TAGS file. Example: '("*.java" "*.jsp")</td>
</tr>
<tr>
  <td>ignore-patterns</td>
  <td>Optional</td>
  <td>List of shell patterns to avoid searching for with <code>project-find-file</code> and <code>project-grep</code>. Example: '("*.class")</td>
</tr>
<tr>
  <td>ack-args</td>
  <td>Optional</td>
  <td>String of arguments to pass to the ack command. Example: "--java"</td>
</tr>
<tr>
  <td>vcs</td>
  <td>Optional</td>
  <td>When set to one of 'git, 'cvs, 'svn, 'bzr, 'hg, or 'darcs the grep and index commands will ignore the VCS's private files 
  (for example, the .git directory or the .CVS directories).</td>
</tr>
<tr>
  <td>tags-file</td>
  <td>Optional</td>
  <td>Path to the TAGS file for this project. Use an absolute path, not one relative to basedir. Value is expanded with <code>expand-file-name</code>.</td>
</tr>
<tr>
  <td>compile-cmd</td>
  <td>Optional</td>
  <td>Command to build the project. Can be either a string specifying a shell command or the name of a function to call. Example: "make -k"</td>
</tr>
<tr>
  <td>startup-hook</td>
  <td>Optional</td>
  <td>Hook function to run after the project is loaded. Project variables (e.g. <i>mk-proj-basedir</i>) will be set and can be referenced from this function.</td>
</tr>
<tr>
  <td>shutdown-hook</td>
  <td>Optional</td>
  <td>Hook function to run after the project is unloaded. Project variables (e.g. <i>mk-proj-basedir</i>) will still be set and can be referenced from this function.</td>
  <td></td>
</tr>
<tr>
  <td>file-list-cache</td>
  <td>Optional</td>
  <td>Cache *file-index* buffer to this file. If set, the *file-index* buffer will take its initial value from this file and updates to the buffer via <code>project-index</code>
  will save to this file. Value is expanded with <code>expand-file-name</code>.</td>
</tr>
<tr>
  <td>open-files-cache</td>
  <td>Optional</td>
  <td>Cache the names of open project files in this file. If set, <code>project-load</code> will open all files listed in this file and <code>project-unload</code> will write all open project 
  files to this file. Value is expanded with <code>expand-file-name</code>.</td>
</tr>
</table>

The following advanced directives are useful if the 'find' commands generated by <code>project-tags</code>, <code>project-grep</code> or <code>project-index</code> aren't sophisticated enough for your project. For example, if you would like to exclude a particular directory from your TAGS file, you can write a custom find command which uses the "-prune" option to exclude the path.

<table border="1">
<tr>
  <th>Directive</th>
  <th>Required/Optional</th>
  <th>Description</th>
</tr>
<tr>
  <td>src-find-cmd</td>
  <td>Optional</td>
  <td><p>Specifies a custom "find" command to locate all files to be included in the TAGS file (see <code>project-tags</code>). 
  The value is either a string or a function of one argument that returns a string. The argument to the function will be the symbol 'src.</p>
  
  <p>
  Examples:<br>
  <pre>
  <code>
  ;; A simple string with a custom find command. However, if your
  ;; command is this simple, you don't need <i>src-find-cmd</i> -- just
  ;; use <i>src-patterns</i>
  (src-find-cmd "find /home/me/my-java-project/ -type f -name '*.java'")
  </code>
  </pre>
  
  <pre>
  <code>
  ;; an anonymous function that builds a custom find command
  (src-find-cmd (lambda (context)
                  (concat "find " mk-proj-basedir " -type f -name '*java'")))
  </code>
  </pre>

  <pre>
  <code>
  ;; a single function that returns a more complex find command based on context
  (defun my-find-cmd (context)
    (let* ((ignore-clause  (concat "\\( -path " mk-proj-basedir "/thirdparty -prune \\)"))
           (src-clause     "\\( -type f \\( -name '*.cpp' -o -name '*.[cChH]' -o -name '*.java' \\) -print \\)"))
      (ecase context
        ('src   (concat "find " mk-proj-basedir " " ignore-clause " -o " src-clause))
        ('grep  (replace-regexp-in-string "-print" "-print0" (concat "find . " src-clause) t))
        ('index (concat "find " mk-proj-basedir " " ignore-clause " -o -print")))))
        
  (src-find-cmd   my-find-cmd)
  (index-find-cmd my-find-cmd)
  (grep-find-cmd  my-find-cmd)
  </code>
  </pre>
  </p>
  
  <p>If non-null (or if the function returns non-null), the custom find command will be used and the 
  <i>src-patterns</i> and <i>vcs</i> settings are ignored when finding files to include in TAGS.</p> </td>
</tr>
<tr>
  <td>grep-find-cmd</td>
  <td>Optional</td>
  <td>
  
  <p>Specifies a custom "find" command to use as the default when
running <code>project-grep</code>. The value is either a string or
a function of one argument that returns a string. The argument to
the function will be the symbol 'grep. The string or returned
string MUST use find's "-print0" argument as the results of
this command are piped to "xargs -0 ...".</p>

<p>If non-null (or if the function returns non-null), the custom
find command will be used and the <i>ignore-patterns</i> and
<i>vcs</i> settings will not be used in the grep command.</p>

<p>The custom find command should use "." (current directory) as
the path that find starts at -- this will allow the C-u argument
to <code>project-grep</code> to run the command from the current buffer's
directory.</p>

<p>See the documentation of <i>src-find-cmd</i> for example values.</p> </td>
</tr>
<tr>
  <td>index-find-cmd</td>
  <td>Optional</td>
  <td><p>Specifies a custom "find" command to use when building an
listing of all files in the project (to be used by
<code>project-find-file</code>). The value is either a string or a
function of one argument that returns a string. The argument to
the function will be the symbol 'index.</p>

<p>If non-null (or if the function returns non-null), the custom
find command will be used and the <i>ignore-patterns</i> and
<i>vcs</i> settings are not used when in the grep command.</p>

<p>See the documentation of <i>src-find-cmd</i> for example values.</p></td>
</tr>
</table>


Where can I get it?
-------------------

Releases of mk-project are uploaded to the [Emacs Wiki](http://www.emacswiki.org/cgi-bin/wiki/mk-project.el).

You can track the code at github: http://github.com/mattkeller/mk-project.


How do I install it?
--------------------

Download [mk-project.el](http://www.emacswiki.org/cgi-bin/wiki/mk-project.el). Place it in your emacs load-path. Load the library with <code>(require 'mk-project)</code>. Then define your projects in your .emacs file. 

The following key bindings are useful:

    (global-set-key (kbd "C-c p c") 'project-compile)
    (global-set-key (kbd "C-c p l") 'project-load)
    (global-set-key (kbd "C-c p a") 'project-ack)
    (global-set-key (kbd "C-c p g") 'project-grep)
    (global-set-key (kbd "C-c p o") 'project-multi-occur)
    (global-set-key (kbd "C-c p u") 'project-unload)
    (global-set-key (kbd "C-c p f") 'project-find-file) ; or project-find-file-ido
    (global-set-key (kbd "C-c p i") 'project-index)
    (global-set-key (kbd "C-c p s") 'project-status)
    (global-set-key (kbd "C-c p h") 'project-home)
    (global-set-key (kbd "C-c p d") 'project-dired)
    (global-set-key (kbd "C-c p t") 'project-tags)


What's going on with it?
------------------------

* 2010/10/23: Released version 1.5.1 with [some new features](http://www.littleredbat.net/mk/blog/story/86).
* 2010/04/18: Released version 1.4.0 with [several enhancements](http://www.littleredbat.net/mk/blog/story/84/).
* 2010/02/09: Released version 1.3.1 with a [default-directory bug fix](http://www.littleredbat.net/mk/blog/story/83).
* 2010/01/30: Released version 1.3 with [custom find command](http://www.littleredbat.net/mk/blog/story/82) support.
* 2009/12/16: Released version 1.2.1 with a [bug fix](http://www.littleredbat.net/mk/blog/story/81/).
* 2009/09/09: Released version 1.2 with [ido](http://www.emacswiki.org/emacs/InteractivelyDoThings) integration.
* 2009/06/11: Released version 1.1 [with ack support](http://www.littleredbat.net/mk/blog/story/77/).
* 2009/05/26: Pushed mk-project v1.0.3 to [github](http://github.com/mattkeller/mk-project/tree/1.0.3).
* 2009/05/26: Released [version 1.0.2](http://www.littleredbat.net/mk/blog/story/75/).
* 2009/05/23: Released [version 1.0.1](http://www.littleredbat.net/mk/blog/story/74/).

Why is it?
----------

There are several Emacs libraries that do similar things ([ProjMan](http://www.emacswiki.org/emacs/ProjmanMode), [project-root](http://www.shellarchive.co.uk/content/lisp/project-root.el)), but none of them fit all my needs. Uniquely, mk-project.el allows users to define projects in pure elisp while most of the existing projects required per-directory config files which don't always fit well with your version control software. Specifically, the libraries I mentioned do not work well with Clearcase's dynamic views.


Found a bug or have a feature request?
--------------------------------------

Feel free to contact Matt at mk -at- littleredbat -dot- net or [submit an issue online](http://github.com/mattkeller/mk-project/issues).


How is it licensed?
-------------------

mk-project is licensed under the [GPLv2](http://www.gnu.org/licenses/gpl-2.0.html).


Who wrote it?
-------------

[Matt Keller](http://www.littleredbat.net/mk) is the primary author.

[Andreas Raster](http://github.com/rakete) contributed code and ideas to the 1.5 release. Andreas also maintains a ['anything' integration module](http://github.com/rakete/mk-project/blob/master/mk-project-anything.el).

Various others have contributed bug reports and ideas. 

Thanks to all for their code and ideas!

