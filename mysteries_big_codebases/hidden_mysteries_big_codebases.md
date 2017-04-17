theme: Fira, 2
autoscale: true
build-lists: true

![](img/background.jpg)

## hidden mysteries behind **big mobile codebases.**

#### <br>@fernando_cejas

^ You have a really cool and impactful project, but as soon as your codebase gets bigger, and more and more contributors come into play, things can become challenging in regards to aspects like: code consistency, technical debt, refactoring, application architecture and team organization. Let's jump onboard on this journey and walk through different techniques that can help us keep our code sane and healthy for better scalability. 
Disclaimer: This talk is going to be focused from a mobile standpoint but most of the practices included can also be applied to any software project under development. 	

---

![](img/background.jpg)

# **Meet @fernando_cejas**

- __*Curious learner*__<br>
- __*Software engineer*__<br>
- __*Speaker*__<br>
- __*Work at @soundcloud*__<br>
- __*fernandocejas.com*__

^ When I really have something to say, I either create a presentation or write a post.
^ Most valuable information comes from experiences.
^ This talk is basically to avoid you guys to bang your head against a brick wall as I did in the past.

---

![](img/background.jpg)

# **This begins with a story...**
 
- Jon was a happy developer
- He had a lightweight pet project
- He was the only maintainer

^ The project was like a baby for him.

---

![](img/background.jpg)

# [fit] **One man Development Process Model.**

^ Easy to maintain.
^ Technical decisions are made by only one person.
^ No conflicts on the code.

---

![](img/background.jpg)

# **At some point in time...**

- Project started to grow...
- More features were required...
- Jon was very happy for its success...

^ Someone believed in that project.

---

![](img/background.jpg)

# **First problem: Success!**

...more and more users using the application. 

---

![](img/background.jpg)

# **Second problem: Success!**

- Code started to grow.
- No tests.
- Inconsistency across the codebase.

---

![](img/background.jpg)

# **Unsustainable situation. Why?**

- More requirements/features.
- More contributors.
- Time to market and dealines.
- Complexity going up.

---

![](img/background.jpg)

# **Many questions to answer.**

- Can we add a new functionality fast?
- Is our codebase prepare to scale?
- Is it hard to maintain? 
- What about technical debt?
- How to keep it healthy and sane?
- Is it easy to onboard new people?
- What about our team organization?

---

![](img/background.jpg)

# Fact **#1**

_**If your codebase is hard to work with...then change it!**_

---

![](img/background.jpg)

# **Soundcloud**

- From a monolith to a microservices architecture.

---

![](img/background.jpg)

# [fit] **Soundcloud Listeners app repo.**

---

![](img/background.jpg)

## **What can we do in terms of...** 

- Codebase.
- Team Organization.
- Working culture.
- Processes.

## **...to support big mobile code bases?[^1]**

[^1]: Disclaimer: no silver bullets.

--- 

![](img/background.jpg)

# [fit] **Our Codebase and its worst enemies...**

---	

# **Size** 
<br>

```java
	Methods in app-dev-debug.apk: 95586 
	Fields in app-dev-debug.apk: 61738 
	Lines of code: 137387
```

---

# **Complexity**

```java
					    private Node delete(Node h, Key key) { 
					        // assert get(h, key) != null;

					        if (key.compareTo(h.key) < 0)  {
					            if (!isRed(h.left) && !isRed(h.left.left))
					                h = moveRedLeft(h);
					            h.left = delete(h.left, key);
					        }
					        else {
					            if (isRed(h.left))
					                h = rotateRight(h);
					            if (key.compareTo(h.key) == 0 && (h.right == null))
					                return null;
					            if (!isRed(h.right) && !isRed(h.right.left))
					                h = moveRedRight(h);
					            if (key.compareTo(h.key) == 0) {
					                Node x = min(h.right);
					                h.key = x.key;
					                h.val = x.val;
					                // h.val = get(h.right, min(h.right).key);
					                // h.key = min(h.right).key;
					                h.right = deleteMin(h.right);
					            }
					            else h.right = delete(h.right, key);
					        }
					        return balance(h);
					    }
```

---

# **Flaky tests**

```
+---------------------------------------------------------------+------------------------------------------------------+--------------+
| ClassName                                                     | TestName                                             | FailureCount |
+---------------------------------------------------------------+------------------------------------------------------+--------------+
| com.soundcloud.android.tests.stations.StationHomePageTest     | testOpenStationShouldResume                          |            7 |
| com.soundcloud.android.tests.stream.CardEngagementTest        | testStreamItemActions                                |            4 |
| com.soundcloud.android.tests.stations.RecommendedStationsTest | testOpenSuggestedStationFromDiscovery                |            3 |
| com.soundcloud.android.tests.player.ads.VideoAdsTest          | testQuartileEvents                                   |            2 |
| com.soundcloud.android.tests.player.ads.VideoAdsTest          | testTappingVideoTwiceResumesPlayingAd                |            2 |
| com.soundcloud.android.tests.player.ads.AudioAdTest           | testQuartileEvents                                   |            2 |
+---------------------------------------------------------------+------------------------------------------------------+--------------+
```

---

# **Anti-patterns** 

```java
	public class NotificationImageDownloader extends AsyncTask<String, Void, Bitmap> {
	    private static final int READ_TIMEOUT = 10 * 1000;
	    private static final int CONNECT_TIMEOUT = 10 * 1000;

	    @Override
	    protected Bitmap doInBackground(String... params) {
	        HttpURLConnection connection = null;
	        try {
	            connection = (HttpURLConnection) new URL(params[0]).openConnection();
	            connection.setConnectTimeout(CONNECT_TIMEOUT);
	            connection.setReadTimeout(READ_TIMEOUT);
	            return BitmapFactory.decodeStream(connection.getInputStream());

	        } catch (IOException e) {
	            e.printStackTrace();
	            return null;

	        } finally {
	            if (connection != null) {
	                connection.disconnect();
	            }
	        }
	    }
	}
```

---

# **Technical Debt** 

```java
	public class PublicApi {
	    public static final String LINKED_PARTITIONING = "linked_partitioning";
	    public static final String TAG = PublicApi.class.getSimpleName();

	    public static final int TIMEOUT = 20 * 1000;
	    public static final long KEEPALIVE_TIMEOUT = 20 * 1000;


	    public static final int MAX_TOTAL_CONNECTIONS = 10;

	    private static PublicApi instance;

	    @Deprecated
	    public PublicApi(Context context) {
	        this(context,
	             SoundCloudApplication.fromContext(context).getAccountOperations(),
	             new ApplicationProperties(context.getResources()), new BuildHelper());

	    }

	    @Deprecated
	    public PublicApi(Context context, AccountOperations accountOperations,
	                     ApplicationProperties applicationProperties, BuildHelper buildHelper) {
	        this(context, buildObjectMapper(), new OAuth(accountOperations),
	             accountOperations, applicationProperties,
	             UnauthorisedRequestRegistry.getInstance(context), new DeviceHelper(context, buildHelper, context.getResources()));
	    }

	    public synchronized static PublicApi getInstance(Context context) {
	        if (instance == null) {
	            instance = new PublicApi(context.getApplicationContext());
	        }
	        return instance;
	    }
	}
```

---

![](img/background.jpg)

# [fit] **How can we battle this enemies and**
# [fit] **conquer a large mobile code base?**

---

![](img/background.jpg)

# Fact **#2**

_**Architecture matters:**_

- New requirements require a new architecture.
- Scalability requires a new architecture.

---

![](img/background.jpg)

# [fit] **Pick an architecture and stick to it**

- Onion Layers
- Clean Architecture
- Ports and adapters
- Model View Presenter
- Custom combination
- Your own
- Sacrificial Architecture?

---

![](img/background.jpg)

# **Benefits of a good architecture:**

- Rapid development.
- Good Scalability.
- Consistency across the codebase.

---

# **Architecture**

```java
												public class MainActivity extends PlayerActivity {

												    @Inject PlaySessionController playSessionController;
												    @Inject Navigator navigator;
												    @Inject FeatureFlags featureFlags;

												    @Inject @LightCycle MainTabsPresenter mainPresenter;
												    @Inject @LightCycle GcmManager gcmManager;
												    @Inject @LightCycle FacebookInvitesController facebookInvitesController;

												    public MainActivity() {
												        SoundCloudApplication.getObjectGraph().inject(this);
												    }

												    protected void onCreate(Bundle savedInstanceState) {
												        redirectToResolverIfNecessary(getIntent());
												        super.onCreate(savedInstanceState);

												        if (savedInstanceState == null) {
												            playSessionController.reloadQueueAndShowPlayerIfEmpty();
												        }
												    }

												    @Override
												    protected void setActivityContentView() {
												        mainPresenter.setBaseLayout(this);
												    }

												    @Override
												    protected void onNewIntent(Intent intent) {
												        redirectToResolverIfNecessary(intent);
												        super.onNewIntent(intent);
												        setIntent(intent);
												    }

												    private void redirectToResolverIfNecessary(Intent intent) {
												        final Uri data = intent.getData();
												        if (data != null
												                && ResolveActivity.accept(data, getResources())
												                && !NavigationIntentHelper.resolvesToNavigationItem(data)) {
												            redirectFacebookDeeplinkToResolver(data);
												        }
												    }

												    private void redirectFacebookDeeplinkToResolver(Uri data) {
												        startActivity(new Intent(this, ResolveActivity.class).setAction(Intent.ACTION_VIEW).setData(data));
												        finish();
												    }
												}
```

---

# **Architecture**

```java
			public class StreamFragment extends LightCycleSupportFragment<StreamFragment>
			        implements RefreshableScreen, ScrollContent {

			    @Inject @LightCycle StreamPresenter presenter;

			    public StreamFragment() {
			        setRetainInstance(true);
			        SoundCloudApplication.getObjectGraph().inject(this);
			    }

			    @Override
			    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
			        return inflater.inflate(getLayoutResource(), container, false);
			    }

			    @Override
			    public MultiSwipeRefreshLayout getRefreshLayout() {
			        return (MultiSwipeRefreshLayout) getView().findViewById(R.id.str_layout);
			    }

			    @Override
			    public View[] getRefreshableViews() {
			        return new View[]{presenter.getRecyclerView(), presenter.getEmptyView()};
			    }

			    @Override
			    public void resetScroll() {
			        presenter.scrollToTop();
			    }

			    private int getLayoutResource() {
			        return R.layout.recyclerview_with_refresh_and_page_bg;
			    }
			}
```

---

![](img/background.jpg)

# Fact **#3**

_**Code evolution implies:**_


- Constant refactoring.
- Exploring new technologies.
- Taking new approaches.

---

![](img/background.jpg)

# **Refactoring**

- Code evolution.
- Boy scouting.
- Step by Step.

---

# **Code to refactor:**

```java
	private void startProcessing(Map<MyKeyEnum, String> map) {
	    Processor myProcessor = new Processor();
	    for (Entry entry : map.entrySet()) {
	        switch(entry.getKey()) {
	            case KEY1:
	                myProcessor.processStuffAboutKey1(entry.getValue());
	                break;
	            case KEY2:
	                myProcessor.processStuffAboutKey2(entry.getValue());
	                break;
	            case KEY3:
	                myProcessor.processStuffAboutKey3(entry.getValue());
	                break;
	            case KEY4:
	                myProcessor.processStuffAboutKey4(entry.getValue());
	                break;
	            ...
	            ...
	        }
	    }
	}
```

---

# **Create an abstraction:**

```java
	public interface KeyProcessor {
	    void processStuff(String data);
	}
```

---

# **Fill a map with implementation:**

```java
	Map<Key, KeyProcessor> processors = new HashMap<>();
	processors.add(key1, new Key1Processor());
	...
	...
	processors.add(key4, new Key2Processor());
```

---

# **Use the map in the loop:**

```java
	for (Entry<Key, String> entry: map.entrySet()) {
	    Key key = entry.getKey();
	    KeyProcessor keyProcessor = processors.get(key);
	    if (keyProcessor == null) {
	        throw new IllegalStateException("Unknown processor for key " + key);
	    }
	    final String value = entry.getValue();
	    keyProcessor.processStuff(value);
	}
```

---

![](img/background.jpg)

# Fact **#4**

_**Rely on a good test battery that backs you up.**_

---

![](img/background.jpg)

# **Technical debt**

### "Indebted code is any code that is hard to scan." 
### "Technical debt is anything that increases the difficulty of reading code."

- Anti-patterns.
- Legacy code.
- Abandoned code.
- Code without tests.

---

![](img/background.jpg)

# Fact **#5**

_**Do not let technical debt beat you.**_

---

![](img/background.jpg)

# [fit] **Addressing and detecting technical debt:**

- Technical Debt Radar.
- Static Analysis tools.

---

![](img/background.jpg)

# Fact **#6**

_**Favor code readability over performance unless the last one is critical for your business.**_

---

![](img/background.jpg)

# **Perfomance**

- First rule: Always measure.
- Encapsulate complexity.
- Monitor it.

---

![](img/background.jpg)

# Fact **#7**

_**Share logic and common functionality accross applications.**_

---

![](img/background.jpg)

# **At SoundCloud**

- Android-kit.
- Skippy.
- Lightcycle.
- Propeller.

---

![](img/background.jpg)

# Fact **#8**

_**Automate all the things!**_

---

![](img/background.jpg)

# **At SoundCloud**

- Continuous building.
- Continuous integration.
- Continuous deployment. 

---

![](img/background.jpg)

# **Lessons learned so far:**

- Wrap third party libraries.
- Do not overthink too much and iterate.
- Early optimization is bad.
- Trial/error does not always work.
- Divide and conquer.
- Prevention is better than cure.

---

![](img/background.jpg)

# Fact **#9**

_**Work as a team.**_

---

![](img/background.jpg)

# **Team Organization**

- Platform tech lead
- Core team
- Feature teams
- Testing engineering team

---

![](img/background.jpg)

# **Working culture**

- Pair programming.
- Git branching model.
- Share knowledge with other platforms.
- Agile and flexible.
- Collective Sync meeting.

---

![](img/background.jpg)

# Fact **#10**

_**We work with people, computers are means to reach out to people.**_

---

![](img/background.jpg)

# **Processes**

- Onboarding new people.
- Hiring people.
- Sheriff.
- Releasing: Release train model + release captains.
- Alpha for internal use.
- Beta community.

---

![](img/background.jpg)

# **Recap #1**

- **#1** If your codebase is hard to work with, just change it.
- **#2** Architecture matters.
- **#3** Code evolution implies continuous improvement. 
- **#4** Rely on a good test battery that backs you up.
- **#5** Do not let Technical Debt beat you.

---

![](img/background.jpg)

# **Recap #2**

- **#6** Favor code readability over performance, unless it is critical.
- **#7** Share logic and common functionality accross applications.
- **#8** Automate all the things.
- **#9** Work as a team.
- **#10** We work with people, computers are means to reach out people.

---

![](img/background.jpg)

# **Conclusion**

- Use **S.O.L.I.D**
- Software development is a **joyful ride.**
- Make it **fun.**

---

![](img/background.jpg)

# **Q & A**

---

![](img/background.jpg)

# **Thanks!!!**
<br>

- __*@fernando_cejas*__<br>
- __*fernandocejas.com*__<br>
- __*soundcloud.com/jobs*__<br>