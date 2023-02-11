---
author: udinic
comments: true
date: 2011-08-17 22:42:29+00:00
layout: post
slug: dress-up-your-preferenceactivity
title: Dress up your PreferenceActivity
wordpress_id: 65
categories:
- Preferences
- Themes
tags:
- Android
- customize
- theme
---

Does your Android app support themes?

Even if the answer is "No", lots of apps are built using colors other than the default black. For instance, using a white background for all the screens. In that case, you're getting yourself into a journey of properties changing, widgets extending and XML modifying. BUT, it's not as difficult as you might think! Sure, there's a need to extend a UI component once in a while, just because it doesn't have a "setBackgroundColor()" or something like that, but besides that - it's mostly just XML attributes needed to be adjusted.

I had the chance to write that kind of app, and I ran into quite a lot of issues in the process, and luckily for you - I'm here to share! I'll start talking about the PreferenceActivity, and if there will be requests - I'll add more posts on that subject.

<!--break-->

If you'll try to add a PreferenceActivity to your app, you'll see that the default background is black. The PreferenceActivity contains a ListView for all the properties inside. We can just get the list from the activity, and change it's background color. That's how we'll do it:

{% highlight java %}
public class UdinicPreferenceActivity extends PreferenceActivity
{
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        addPreferencesFromResource(R.xml.udinic_pref);

        // Setting the list's background to be white
        findViewById(android.R.id.list).setBackgroundColor(Color.WHITE);
    }
}
{% endhighlight %}

The background has indeed been changed. BUT, the preferences' categories still have black background, as you can see in this screen-shot:

[![](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/1.png?w=180)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/1.png)

A thorough review of its XML attributes, reveals that the PreferenceCategory doesn't have an attributes for the background color. After thinking a little, I decided to override the onCreateView(), which comes from the Preference object (as our PreferenceCategory). This function gets called in order to create the preference's View object, that's why it's important to call the super before anything we're trying to do here. The new PreferenceCategory class should be used in the preference xml, instead of the default one. This is the complete customized PreferenceCategory that I've created to solve this issue:

{% highlight java %}
public class UdinicPreferenceCategory extends PreferenceCategory {

	public UdinicPreferenceCategory(Context context) {
		super(context);
	}

	public UdinicPreferenceCategory(Context context, AttributeSet attrs) {
		super(context, attrs);
	}

	public UdinicPreferenceCategory(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
	}

    /**
     * We catch the view after its creation, and before the activity will use it, in order to make our changes
     * @param parent
     * @return
     */
	@Override
	protected View onCreateView(ViewGroup parent) {
        // And it's just a TextView!
		TextView categoryTitle =  (TextView)super.onCreateView(parent);
		categoryTitle.setBackgroundColor(Color.WHITE);
		categoryTitle.setTextColor(Color.RED);

        return categoryTitle;
	}
}
{% endhighlight %}

We are changing the background and also the text color, making the activity more...festive. This is the result:

[![](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/2.png?w=180)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/2.png)

And now you have a pretty and customized PreferenceActivity! I'm afraid I wasn't completely honest with you, the Properties in the screenshots are also customized (The default text color is white, how else could you see it on top a white background?!), but I used the same technique as for the PreferenceCategory. The full source code is in my [github repository](https://github.com/Udinic/SmallExamples).

Please comment me if you want to learn more about customizing your application's colors, and there are more issues about that. For instance, in this app - try to scroll through the Preference list, and see a problem that needs to be solved :)
