=== Edit Comments ===

Tags: comments,edit comments
Contributors: jalenack
This release: August 27, 2005
Version: 0.11 Alpha

==== WARNING ====

This version of the plugin is ALPHA. That means its for people who are very comfortable with code. As the code matures and I determine that its ready for mainstream use, I will remove this warning.

== Description ==

Edit Comments is a simple WordPress plugin that allows commenters to edit their own comments. To edit a comment, a user must have the same IP address as the user that made the comment and they must also make the edit within a specific time frame. The default edit time window is 30 minutes, but it can be changed easily in the plugin file.

== Installation ==

Unfortunately, the edits required to make this plugin work are fairly extensive. They don't, however, require any core edits, which means upgrading WordPress versions will not affect this plugin. I'll try to explain the edits the best I can.

1. Upload the plugin file "edit-comments.php" to your plugins folder. This folder is located at /wp-contents/plugins/

2. Activate the plugin from your WordPress admin panel.

3. Here's where it gets tricky. We're going to need to make some changes to the comments.php file of your theme. This file is located in /wp-content/themes/**YOUR THEME NAME***/comments.php

4. Before we make any edits, let's make a backup of comments.php . Copy the file and name it comments-backup.php . You should now have two comment files in your theme directory, one called comments-backup.php and one called comments.php .

NOTE: For people using the default theme, Kubrick, I've included a modified copy of the comments.php file. It contains all the edits needed to make the plugin work. If you're using a Kubrick-based theme, that file may also work for you.

5. For the rest of you, it'll require quite a bit of tweaking to get it right. Make the following edits to your comments.php file:

5a. First thing to do is add the link that users will click on to edit their comments. This will be added within the comment loop. The beginning of the comment loop looks like this:

<?php foreach ($comments as $comment) : ?>

And the end of the comment loop looks like:

<?php endforeach; /* end for each comment */ ?>

Anything inbetween those two bits of code will be repeated for each comment. We need to add a tag that will create the link somewhere in this loop. The code is: <?php jal_edit_comment(); ?>. An example would be:

<small class="commentmetadata"><?php comment_date('F jS, Y') ?> at <?php comment_time() ?> | <?php jal_edit_comment(); ?></small>

This will output the comment date and time, and then the edit link.

5b. Now we need to account for three possible scenarios.

* Someone who is logged in and making a comment
* Someone who isn't logged in and making a comment
* Someone who is editing their comment (logged in or not doesn't matter)

In your file, the first 2 are already accounted for. We need to make room for the last one.

Find this: <?php if ( $user_ID ) : ?> . It means there is someone logged in. The next line might be something like "<p>Logged in as....."

Here we go with a big edit. Delete this: <?php if ( $user_ID ) : ?>
And change it to:

 <?php if ( isset($_GET['jal_edit_comments']) ) : 
 	global $jal_minutes;
 	$jal_comment = $wpdb->get_row("SELECT comment_content, comment_author_IP, comment_date FROM $wpdb->comments WHERE comment_ID = ".$_GET['jal_edit_comments']);
 	$time_ago = time() - strtotime($jal_comment->comment_date);
 	
	// security doesn't matter now. Checked again during the edit
	echo '<p><input type="hidden" name="jal_edit_this" value="'.$_GET['jal_edit_comments'].'" /></p>';


 	if ($_SERVER['REMOTE_ADDR'] != $jal_comment->comment_author_IP || $time_ago >= 60 * $jal_minutes) :
 		echo "<p><strong>You aren't allowed to edit this comment, either because you didn't write it or you passed the {$jal_minutes} minute time limit.</strong></p>";
 		echo "</form>";
 		// get out of this template
 		return;
 	endif;
 ?>
 
 <?php elseif ( $user_ID ) : ?>

5c. Now we need to edit the <textarea> where users input their comments. Look for:

<textarea name="comment" id="comment" cols="100%" rows="10" tabindex="4"></textarea>

And replace it with: 	

 <textarea name="comment" id="comment" cols="100%" rows="10" tabindex="4"><?php

	if ($jal_comment) echo $jal_comment->comment_content;	

 ?></textarea>

5d. You're pretty much done! Now, we can just do a bit of beautification and cleanup.

We obviously don't want to have the "Leave a Reply" message as the header of the comment area if they're editing a comment. So change this:

<h3 id="respond">Leave a Reply</h3>

To:

<h3 id="respond"><?php echo (isset($_GET['jal_edit_comments'])) ? "Edit your Comment" : "Leave a Reply"; ?></h3>

The above uses a ternary operator. Basically, if the someone is editing a comment, it will print "Edit your Comment", and if not it will print "Leave a Reply" . You could also use this technique for the submit button.

6. Break out a bottle of champagne. You just finished the installation of Edit Comments. Enjoy!

== FAQs ==

= How does it work? =

WordPress logs the IP address of all commenters, so we can compare the IP address of the comment to the IP address of the viewer. The plugin also checks the timestamp of the comment with the current time. If more than 30 minutes (default) has passed since the comment was made, then editing is off-limits. These two checks together make it secure.

= Why is there a time limit on editing? =

This plugin is useful when a user hits "Submit Comment" and then realizes he or she made a glaring spelling error, or forgot an important part of what they were going to say.

Also, by having a time limit, users aren't allowed to go back and change something they said days ago that they now regret. When there's a discussion, its best if people can't go back on their words, or else it can get really confusing.

Lastly, it's good for security. In the extremely unlikely case that someone   else gets the same IP address as one of your commenters and then goes and visits your blog, they won't be able to edit the comment because of the time-limit.

= How can I customize jal_edit_comment() ? =

This function takes 4 arguments.
jal_edit_comment ( $text, $before, $after, $editing_message )

$text is the text that the link will show for. Default is 'edit'

$before is for adding code before the link. The default is none

$after is for adding code after the link. The default is none

$editing_message is for when users have clicked the edit link. This message will appear in the comment they are editing. Default is '<strong>EDITING</strong>'

Example: 

jal_edit_comment("(e)", "<strong>", "</strong>", "<em>(editing)</em>");

Will output: <strong><a href="...edit_link url">(e)</a></strong> .

= I am totally lost on how to install this. Can you do it for me? =

Yeah, I guess. Email me with your comments.php file attached, and I'll try to make the edits for you. No promises though. A small donation would go a long way in motivating me to do it for you :)

== Contact Developer ==

I am always eager to help you with your problems involving this plugin. Chances are there's something wrong with my code and by telling me you are probably helping other users too.

* Email:
	jalenack@gmail.com
* AIM:
	Harc Serf
ICQ:
	305917227
Yahoo:
	jalenack
MSN Messenger:
	jalenack@hotmail.com
* Online Contact form:
	http://blog.jalenack.com/contact.php

* = Most likely venues for a quick response