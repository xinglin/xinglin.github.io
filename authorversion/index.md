# Create The Author Version for Your Paper (ACM)


So you have your paper accepted to a conference and completed your camera ready version. Now is the time to create an author version for it so that you can post it on your own web site! The trick is actually quite simple if you are using the ACM Latex template.

In acmart.cls, change the execution option for `authorversion` from false to true. 
<pre>
-\ExecuteOptionsX{authorversion=false}
+\ExecuteOptionsX{authorversion=true}
</pre>

You may also want to add page numbers into your paper. 
<pre>
    \pagestyle{plain}
    \pagenumbering{arabic}
</pre>
