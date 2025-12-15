**Jonathan Mackey > Miscellaneous**
-----------------------------------

Miscellaneous computing stuff
---------------------------------------------------------

* [Batch job submission with SLURM](slurm.html "slurm.html"): list of useful commands to help figure out what is going on with your jobs on a supercomputer.
* [SSH](#ssh "#ssh"): using SSH to login remotely.
* [Shell scripts](#scripts "#scripts"): useful bits and pieces for bash scripting.
* [Version Control with Mercurial](mercurial.html "mercurial.html"): using Mercurial to keep track of code/docs (links to new page).
* [Imagemagick](#convert "#convert"): How to use the command-line tool 'convert'.
* [Useful Unix commands](#unix "#unix"): random useful commands I keep forgetting how to call.
* [LaTeX](#latex "#latex"): a few hints for LaTeX documents.
* [Gnuplot:](#gnuplot "#gnuplot") some common tasks for astrophysicists using gnuplot.
* [Mac OS X](#mac "#mac") hints and tricks.




SSH tips
--------

* **ssh config file**  
  Here is an example config file in ~/.ssh/config:

  ```
    Host *
            ServerAliveInterval 60

          Host gateway
            HostName gateway.domain.com
            User uname1

          Host remote
            HostName remote.domain.com
            ProxyJump gateway
            User uname2
  ```

  The first two lines set the computer to send a keep-alive signal every 60 seconds to the remote host.
  This can be useful if the remote computer keeps disconnecting you.
    
  The second entry creates an alias for logging in to gateway.domain.com with username uname1.
  This means that instead of typing

  ```
  ssh uname1@gateway.domain.com
  ```

  you can just type

  ```
  ssh gateway
  ```

  This is nice, but then the third entry is where it gets really useful.
  This is for a computer called remote that is behind a firewall and can only be accessed through gateway.
  Usually we would have to type in something like:

  ```
  ssh -XY -t uname1@gateway.domain.com "ssh -XY -t uname2@remote.domain.com bash"
  ```

  With the ssh config file, however, you can just type:

  ```
  ssh -XY remote
  ```

  and it is equivalent to the previous command.
  If you need to use a non-standard ssh port, then you can specify a different port number.
    
  See [this tutorial](http://sshmenu.sourceforge.net/articles/transparent-mulithop.html "http://sshmenu.sourceforge.net/articles/transparent-mulithop.html") for further details.

* **Setting up a tunnel back to the local machine**  
  This is useful if you connect to a computer that doesn't allow any outgoing ssh connections to the outside world (only incoming connections).
  When you login to the remote computer (remote.remotedomain.com, username remoteuser), you can open a tunnel to the local computer (local.domain.com, username localuser, port 1234) as follows:

  ```
  ssh -XY -l remoteuser -R 54321:localuser@local.domain.com:1234 remote.remotedomain.com
  ```

  Once you are logged in to the remote computer, you can connect back to the local computer using:

  ```
  ssh localuser@localhost:22424
  ```

* **using FUSE to mount remote filesystems**  
  SSHFS is a nice package for accessing remote filesystems on a local computer.
  You can install it (on debian/ubuntu linux) with

  ```
  aptitude install sshfs
  ```

  This will also install fuse, which is what does most of the work.
    
  First choose where you want to mount the filesystem, and create a directory.
  I do this as root, but then change the owner of the mount directory to the user, here "jm".
  You also need to add jm to the fuse group, so that jm has permission to mount remote filesystems

  ```
    mkdir /mnt/remote
          chown jm:users /mnt/remote/
          adduser jm fuse
  ```

  Then you can mount the remote filesystem (remote.domain.com, username remoteuser) as local user jm with:

  ```
    sshfs -o idmap=user remoteuser@remote.domain.com:/path/to/mount/point/ /mnt/remote/
  ```

  The command idmap=user maps the remoteuser to jm, so that it looks like jm owns all the files.
  The remote filesystem can be unmounted with the command:

  ```
    fusermount -u /mnt/remote/
  ```

  There are lots of online tutorials for this, especially to help with permissions, username mapping, and fuse:
  [Ubuntu help](https://help.ubuntu.com/community/SSHFS "https://help.ubuntu.com/community/SSHFS"),
  [SSHFS on github](https://github.com/libfuse/sshfs "https://github.com/libfuse/sshfs"),
  [debianadmin.com](http://www.debianadmin.com/mount-a-remote-file-system-through-ssh-using-sshfs.html "http://www.debianadmin.com/mount-a-remote-file-system-through-ssh-using-sshfs.html")





Shell scripts
-------------

* **Deleting text between two identifiers in a file**  
  (found [here](http://www.cyberciti.biz/faq/sed-howto-remove-lines-paragraphs/ "http://www.cyberciti.biz/faq/sed-howto-remove-lines-paragraphs/")), which can be useful for generating clean source code without all of the debugging ifdefs.

  ```
  sed '/\#ifdef DEBUGGING/,/\#endif/d' file.cpp > output.cpp
  ```

  This can of course get confused if you have nested ifdefs...
* **Replace text in a variable:**  
  If FILE=hello.png is a PNG image, and you want to convert to an image called hello.eps, then you can do this:

  ```
  IMGFILE=`echo $FILE | sed 's/png/eps/'`
        convert $FILE eps3:$IMGFILE
  ```
* **Fixed number of digits in a filename:**  
  If you are generating a sequence of files, and want them numbered with a counter with exactly N digits, you can use this:

  ```
        $ BASE=myfilename
        $ for i in `seq 0 10`; do
        >   FILE=`printf "%s_%03d.tif" $BASE $i`; echo $FILE
        > done
        myfilename_000.tif
        myfilename_001.tif
        myfilename_002.tif
        myfilename_003.tif
        myfilename_004.tif
        myfilename_005.tif
        myfilename_006.tif
        myfilename_007.tif
        myfilename_008.tif
        myfilename_009.tif
        myfilename_010.tif
  ```
* **Multiplying floating point numbers:**  
  unfortunately bash and other shell scripts can only do integer arithmetic, so for floating point operations you need to use another tool, such as 'bc'. The following line divides 10 by 3.0 using 'bc'; the 'scale=2' command indicates the calculation should be performed to at least two decimal places.

  ```
  $ ii=10; FLT=$(echo "scale=2; $ii/3.0" | bc); echo $FLT
        3.33
  ```





Imagemagick
-----------

[Imagemagick](http://www.imagemagick.org/ "http://www.imagemagick.org/") is a very useful tool for converting figures between formats, cropping borders, shrinking images, etc., and it has a command-line interface so you can put it in scripts. Here are some examples I find useful (most of them taken from the imagemagick website examples).

* **Convert eps to jpeg**  
  (I think density is dots-per-inch, and quality is the jpeg quality flag; density is most important for figures):

  ```
        $ convert -density 300 -quality 100 file.eps file.jpeg
        $ convert -geometry 568x256 -density 300 -quality 100 image.eps image.jpeg
  ```

  The geometry tag sets the size of the output image, and I think it's a good idea to set it to be a multiple or fraction of the eps bounding box (from `head image.eps | grep BoundingBox'). The default (I think) is to set them to be the same.

* **Crop an image**  
  the first two numbers are the number of pixels in the output, and the last two are the x and y offsets from the top left corner to start the cropping from. Putting an exclamation mark at the end of the geometry item resets the new image origin to the top left corner of the new image (otherwise it stays relative to the old one!).

  ```
        $ convert -crop 955x325+175+225 input_file.tif output_file.tif
        $ convert -crop 955x325+175+225! input_file.tif output_file.tif
  ```

* **Annotate an image**  
  pointsize refers to the size of the text; annotate gives the offsets from the top left (of the original image if you cropped it!) to start writing, and the text is in single quotes.

  ```
        $ convert input_file.tif -pointsize 20 -annotate +250+275 \
         'this is an annotation' output_file.tif
  ```
* **Make an image smaller** (by resampling):

  ```
  convert terminal.gif -resize 50% half_terminal.gif
  ```
* **Draw a line** with a certain thickness (coordinates are from the top left in pixels, and refer to the start and end of the line).

  ```
        convert source_file.gif -stroke white -strokewidth 3 -draw 'line 20,20 20,80' output_file.gif
  ```
* **Draw two arrows:** The following lines generate two coordinate axes, with X as horizontal and Z as vertical. The background is white and the foreground is black, although they can of course be changed. The idea is to draw a line, put an arrowhead on the end of it, and position an axis label off the end of each arrow. Positions are measured from the top left by Imagemagick, and positive rotations are in degrees from the X-axis. The triad is then merged onto a figure twice (i.e. in two places) using the "composite" command, which I think is another command-line tool from ImageMagick. It's not a very stylish figure, but it shows what can be done if required.

  ```
        arrow_head="path 'M 0,0  l -15,-5  +5,+5  -5,+5  +15,-5 z'"
        convert -size 275x275 xc:white temp.gif
        convert -size 800x800 xc:green Background.eps

        convert temp.gif -stroke black -strokewidth 10 -draw 'line 30,245 30,075' temp.gif
        convert temp.gif -draw "stroke black fill black translate 30,065 rotate -90 scale 3.5,3.5 $arrow_head " temp.gif
        convert temp.gif -pointsize 72 -font "Times-Roman" -fill black -annotate +9+63 "Z" temp.gif

        convert temp.gif -stroke black -strokewidth 10 -draw 'line 25,245 200,245' temp.gif
        convert temp.gif -draw "stroke black fill black translate 210,245 rotate  0 scale 3.5,3.5 $arrow_head " temp.gif
        convert temp.gif -pointsize 72 -font "Times-Roman" -fill black -annotate +215+270 "X" temp.gif

        convert temp.gif -resize 100x100 eps3:triadXZ_f2.eps
        mv temp.gif axes_triad_XpZp.gif
        composite -geometry +10+575 triadXZ_f2.eps Background.eps temp_fig2.eps
        composite -geometry +10+263 triadXZ_f2.eps temp_fig2.eps temp2_fig2.eps
  ```





Unix Commands
-------------

* **Concatenate pdf files** (found [here](http://doeidoei.wordpress.com/2009/04/12/easy-way-to-concatenate-pdf-files-in-ubuntu-linux/ "http://doeidoei.wordpress.com/2009/04/12/easy-way-to-concatenate-pdf-files-in-ubuntu-linux/")):

  ```
  gs -q -sPAPERSIZE=a4 -dNOPAUSE -dBATCH -sDEVICE=pdfwrite -sOutputFile=output.pdf file1.pdf file2.pdf [...] lastfile.pdf
  ```
* **Delete files with zero size** (found [here](http://www.unix.com/77405-post6.html "http://www.unix.com/77405-post6.html")):

  ```
  find -maxdepth 1 -type f -size 0 -print0 | xargs -0 rm -f
  ```
* **Delete files matching a pattern (e.g. file extension)** (found [here](http://www.cyberciti.biz/faq/linux-unix-how-to-find-and-remove-files/ "http://www.cyberciti.biz/faq/linux-unix-how-to-find-and-remove-files/")):

  ```
  find . -type f -name "FILE-TO-FIND" -exec rm -f {} \;
  ```


LaTeX
-----

* \mathbf{} makes normal bold-face Roman characters, and doesn't allow italics by default. So to get bold italic math characters you can include the following:

  ```
  \DeclareMathAlphabet{\mathitbf}{OML}{cmm}{b}{it}
  ```
* Need the LaTeX name for a symbol, but can't remember it? Draw it here [detexify.kirelabs.org/classify.html](http://detexify.kirelabs.org/classify.html "http://detexify.kirelabs.org/classify.html") and get the LaTeX code!
* Need many sub-figures as part of a single large figure, on multiple pages? Use the subfig class and \ContinuedFloat e.g.

  ```
  % HEADER
  \usepackage{graphicx}
  \usepackage{subfig}
  ...
  % DOCUMENT
  \begin{figure}[h]
    \centering
    \includegraphics[width=0.9\textwidth]{fig1a.eps}
    \includegraphics[width=0.9\textwidth]{fig1b.eps}
    \caption{this is the caption. \label{fig:fig1}}
  \end{figure}

  \begin{figure}
    \ContinuedFloat
    \centering
    \includegraphics[width=0.9\textwidth]{fig1c.eps}
    \includegraphics[width=0.9\textwidth]{fig1d.eps}
    \includegraphics[width=0.9\textwidth]{fig1e.eps}
    \caption{This is the caption (again).}
  \end{figure}
  ```
* Package for striking through text with a horizontal line: [ulem](http://tex.stackexchange.com/questions/23711/strikethrough-text "http://tex.stackexchange.com/questions/23711/strikethrough-text").

  ```
  \usepackage[normalem]{ulem}
  \sout{Hello World}
  ```
* Euro symbol:

  ```
  \usepackage{eurosym}
  ```
* Nicer fonts:

  ```
  \usepackage[T1]{fontenc}
  \usepackage[scaled]{helvet}
  ```
* Less indent and line-spacing in numbered and bullet-point lists:

  ```
  \usepackage{enumitem}
  ...
  \begin{enumerate}[leftmargin=*,itemsep=0pt]
  \item text
  \end{enumerate}
  ```

Mac OSX bits and pieces
-----------------------------------

* **install homebrew** from [brew.sh](https://brew.sh/)

* **sed:**
  Sed works differents on OS X for some reason.
  If you want to replace text in place in a file with e.g.

  ```
  sed -i -e "s/search/replace/" file.txt
  ```

  then on OS X you will get a new file called file.txt-e (as well as the modified file.txt), containing the original version of file.txt.
  This doesn't happen on linux/unix, so to get around it you have to add extra double quotes after the "-i", as follows:

  ```
  sed -i "" -e "s/search/replace/" file.txt
  ```
* **LaTeXiT:**
  [LaTeXiT](http://pierre.chachatelier.fr/latexit/ "http://pierre.chachatelier.fr/latexit/") is a nice program that you can use beside Keynote/Powerpoint as a LaTeX equation editor.
  There is an input box for typing LaTeX commands, and an output box that displays a scalable PDF of the equation that you can then copy and paste directly into Keynote/Powerpoint.





