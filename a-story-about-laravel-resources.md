Leo is a brilliant developer. He turns every idea into code in no time. He’s prepared for any task thrown at him and dives into coding straight away, delivering successfully and rapidly.

These days he’s been working on a public API for the application he and his team is building. A few resources with a few endpoints were not a big challenge for Leo obviously. He caught some models in the wild and turned them into **beatiful REST API** data source. Done deal.

A few days passed and `Whoops! Something went wrong` unfortunately.

But worry not, for Leo is well-prepared – he wrote tests for the API, so whatever went wrong, was caught during proper build process before going live, so all that needs to be done is tracking down the culprit and fixing the issue.

Leo found out that JSON from `GET /users/{uuid}` endpoint returned `posts` related collection, even though it wasn’t requested by the consumer (test in this case).
After scratching his head for 15 minutes, playing with the endpoint manually, checking the transforming method, he was a bit lost.
Finally Leo found out that his colleague, Francis, added a key to the `$appends` array on the `User` model, which in turn called the accessor behind the scenes, and the latter did some calculations on related `posts` collection. This ended up in loading the relation implicitly.

Now, this is a puzzle… Leo had to discuss it with Francis and they realized, that using `$appends` is a very bad idea, but it is too hard now to change it, because features planned for the release are necessary for the business. They agreed, that Leo is gonna make a quick fix in his transformer class and instead of deceptive `whenLoaded` he will simply rely on old-skool, a bit rusty nowadays, yet effective `if` statement.

_Screw it_, he thought to himself, _I’m never gonna show this sh.t on twitter, so let it be…_

Issue fixed, case closed!

---

Things were going great for Leo until one Friday afternoon, when sales team manager Jonathan, approached him.

Leo wasn’t sure whether it was a threatening expression on Jonathan’s face, or it was just a Friday prank.
_He’s serious, too serious_, Leo whispered to his neighbour, Alice, and he was damn right…

The API that Leo created has been working for a few weeks now. It was fine and healthy, there were some inconsistencies in reponse times without any clear reason, but nobody complained.

Today, however, somebody did…

Jonathan was furious about some sensitive users’ data leaking through the API. He was so angry that while shouting he spit on Leo’s shiny MBP, and it wasn’t cool. Not cool at all if you ask me!

It took a while before Leo actually learned from Jonathan what the problem was. Apparently, Jonathan had a friendly meeting with one of the customers, who also was API consumer, and after a drink or two, the customer said something between the lines of

_I’m really glad you guys share all that info there, in your API, but you may want to actually hide it. If I didn’t know you from college I could simply steal your customers, you know… I bet Donnie from XYZ is already doing that mate hahahaha!_

WAT?!

Jonathan thought it was a joke first, but after asking his college friend a few questions, he was horrified and dashed back to the office.

Leo started to panic. He knew, it could end up badly, so he reassured Jonathan that there was no issue in the API and it must have been a glitch that made the guy receive some excess data. He also promised to stay in the office as long as necessary to check everything in and out and fix if there was anything wrong.

He knew, his girlfriend was preparing for a Friday night out already, so he acted quickly. After browsing through all the codes related to `/users` resource in his API, Leo couldn’t find the problem, so he asked Alice for help. They were always there for each other, so she agreed to stay after-hours and help.
Alice wasn’t too familiar with the structure of Resources, so after reading the code, she resorted to checking the docs on the framework’s website.

Spot on! During browsing the docs page, she noticed something familiar: `when($this->isAdmin(), ...)` – they had the same construct in their code. However, both she and Leo missed the important bit in the beginning:

IMAGE

That moment Leo realized he was screwed…

_It isn’t about the authenticated user, it is the requested data where that method is being called_, he muttered, then yelled _FFS!!_

—

The story went on, Leo fixed everything in the API, he asked his old-skool colleague (Dinosaur they called him) Richard for advice this time. He learned it the hard way, but he did, and these days he makes his code predictable.

Now he knows and now he spreads the word out to all the New Kids On The Block:

**Save yourself that Friday Night with your bae mate!**
