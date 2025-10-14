---
author: udinic
comments: true
date: 2011-07-01 09:02:48+00:00
layout: post
slug: selectablelistview-make-selection-work
title: SelectableListView - Make selection work!
wordpress_id: 14
categories:
- Android
- ListView
tags:
- Android
- ListView
- selectable adapter
- selectable list
---

The touch screens brought simplicity to our lives. We can scroll, zoom and click with just a simple gesture, using our fingers. Showing it to me 15 years ago, would've make me think that's an act of pure magic!

However, it brought lots of issues regarding the handling of touch events with other inputs, like keyboard or trackball. With keyboard, we can just press the Up and Down keys to select different items, and afterward hit "Enter" to click on the selected item, making it our "final" pick. How can we do that in a touch screen? If pressing an item on the list perform a "Click" action, how can we perform a "Select" action? The Android OS developers thought about that, and decided to create a mode, called "Touch Mode", which handles focus and selection of items differently. More about it on the [Android Documentation](http://developer.android.com/resources/articles/touch-mode.html). Long story short - they chose to ignore the "selection" of list items in touch mode!

Their decision to make touch interaction simpler for the developer, limits the power of ListView (The old "Simplicity <-> Control" trade-off. Apple is well known for choosing the first approach most of the times). As I needed to make such Selectable-List, I had to find the right way, using the components given to me. The user-experience I was looking for, was that each click on a list item will "select" it, and there will be a different button to press in order to finalize my selection, like a "Set" button under the list itself.

<!--break-->

The idea is simple.



	
  * Use the _setOnItemClickListener_ of ListView to set a "Selected Flag" for the current item

	
  * Refresh the list

	
  * Make sure the selected item is painted differently than the other list items.


The flag representation must be for each item in the list, whether it's a Cursor-Based adapter or an Array-Adapter, we need to make sure to have a field for that information. On a cursor-based adapter, we can add a column to the DB table, making it in charge of that function. In an Array-Adapter, we can make data-structure, Keeping our information aside to a selected-state information. A good example to such data-structure is this:

{% highlight java %}
class StateListItem {
	public String itemTitle;
	public long id;
	public Boolean isItemSelected;

	public StateListItem(String name, long id) {
		this.itemTitle = name;
		this.isItemSelected = false;
		this.id = id;
	}

	@Override
	public String toString() {
		return this.itemTitle;
	}
}
{% endhighlight %}

With the ID, Title and the selected state (_isItemSelected_), we have all the data we need to make this work.

Now, in order to make the list item look different when it's selected, we'll just override the adapter's getView() function, like this:

{% highlight java %}
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	View currView = super.getView(position, convertView, parent);
	StateListItem currItem = getItem(position);
	if (currItem.isItemSelected) {
		currView.setBackgroundColor(Color.RED);
	} else {
		currView.setBackgroundColor(Color.BLACK);
	}
	return currView;
}
{% endhighlight %}

We don't care about the logic of _getView_, we just want to change the view's background to Red when selected, and keep it Black when not. That's why we're calling the _super_'s _getView_ first, and then making our adjustments.
You're probably wondering why do I need to set the default color for each item, and not relying it to be the default color unless I change it to Red. Well, that's due to the ListView's recycling mechanism (Info on that can be found [here](http://www.google.com/events/io/2010/sessions/world-of-listview-android.html)). The ListView is recycling views, which makes us reset any visual property every time we populate them, otherwise - we'll get a strange behavior of the items' appearance.

After knowing how to mark the selected item, and use that to change its looks, we can finish the rest of our customized list adapter, handling the way we want to change and keep the list's current selected item. That's the how it looks:

{% highlight java %}
public class SelectableListAdapter extends ArrayAdapter {

	// Keeping the currently selected item
	int mCurrSelected = -1;

	// Since most of the actions gets the id but needs the position,
	// we'll map Ids to Positions
	private HashMap mIdToPosition;

	public SelectableListAdapter(Context context, int textViewResourceId) {
		super(context, textViewResourceId);
		init();
	}

	private void init() {
		mIdToPosition = new HashMap();
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		View currView = super.getView(position, convertView, parent);
		StateListItem currItem = getItem(position);
		if (currItem.isItemSelected) {
			currView.setBackgroundColor(Color.RED);
		} else {
			currView.setBackgroundColor(Color.BLACK);
		}
		return currView;
	}
	public void add(String object, long id) {
		StateListItem newItem =new StateListItem(object, id);
		// Adding the new item to the Id-&gt;Position HashMap
		mIdToPosition.put(id, getCount());
		super.add(newItem);
	}

	@Override
	public void remove(StateListItem object) {
		super.remove(object);
		mIdToPosition.remove(object.id);
	}

	/**
	 * Setting the item in the argumented position - as selected.
	 * @param position
	 * @return
	 */
	public long setSelectable(int position) {
		// The -1 value means that no item is selected
		if (mCurrSelected != -1) {
			getItem(mCurrSelected).isItemSelected = false;
		}

		// Selecting the item in the position we got as an argument
		if (position != -1) {
			getItem(position).isItemSelected = true;
			mCurrSelected = position;
		}

		// Making the list redraw
		notifyDataSetChanged();

		return getSelectedId();
	}

	public long setSelectableId(long id) {
		// First, we need to get the position for our item's id
		int pos = mIdToPosition.get(id);
		return setSelectable(pos);
	}

	@Override
	public long getItemId(int position) {
		return super.getItem(position).id;
	}

	/**
	 * Needed to notify the ListView's system that the IDs we use here are unique to each item
	 */
	@Override
	public boolean hasStableIds() {
		return true;
	}

	public long getSelectedId() {
		if (mCurrSelected == -1)
			return -1;
		else {
			return getItemId(mCurrSelected);
		}
	}
}
{% endhighlight %}

A cursor-based adapter will be similar to that, just with a different approach to the data - as described before.
This adapter can also be used with any ListView and can easily be adapted to fit ExpandableListView as well!.
