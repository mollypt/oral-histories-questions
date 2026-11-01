<script>
	import { json } from "@sveltejs/kit";
	import data from "$data/narrative-time.csv"
	import metadata from "$data/interview-metadata.csv"
	import { onMount } from 'svelte';
	import { group, bisector } from "d3"
	





</script>
<link href="https://fonts.googleapis.com/css2?family=Barlow:wght@100..900&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://use.typekit.net/gkx8zdh.css">
<link rel="stylesheet" href="https://use.typekit.net/kha5kwj.css">
<link rel="stylesheet" href="https://use.typekit.net/mhl8zjo.css">
<link rel="stylesheet" href="https://s3.amazonaws.com/static.getsnworks.com/fontawesome/5.15.2/css/all.min.css">

<div class="container">hello1!
	<div class="topic-sample">
	<div class="title">Topic keywords: belongings - packing - supplies - suitcases</div>
		<div class="question">How involved were your parents in your packing? Did they pack?</div>
		<div class="question">So the stuff you couldn't carry, what happened to that?</div>
		<div class="question">Can you remember back then what some of your... what some of your favorite possessions might have been?</div>
		<div class="question">So you had a few days to gather up your belongings?</div>
		<div class="question">And what did you pack?</div>
	</div>
	<div class= "side-by-side">
		<div class="side">
			<div class="title">Bathing at Heart Mountain Concentration Camp</div>
			<div class="text"><span class="speaker">Martha Nakagawa:</span>
				Let me ask you a little bit about the <i>latrine</i> situation. What was it like for you to use these public <i>latrines</i> in camp?<br><br> 
				<span class="speaker">Bill Shishima:</span> Oh, that was the worst part of camp life. No privacy, and I think initially, people sort of waited to get late in the evening to go use the bathroom, but I think everyone else had the same idea, so that was the busiest time. So you really had to wait then. So there was approximately eight commodes, four on one side, and small partition, and four more on the other side. And to me it was just too embarrassing. You had to sit next to a stranger and do your personal business. The showers were bad enough. You take a shower in a small room, maybe eight feet by ten feet, there was four showerheads on one wall, four more on the other wall, and eight of you are taking a shower just within arm's reach. That was bad enough, but do your personal business, oh, that was gross.</div>
		</div>
		<div class="side">
			<div class="title">Japanese-American Community Bathhouse in Los Angeles</div>
			<div class="text">
			<span class="speaker">Tom Ikeda:</span> So describe the routine for me. So it's after dinner, about what time would you get ready to go take a <i>bath</i>? 
			<br><br> <span class="speaker">Louie Wantanabe:</span> Well, anytime after maybe eight or nine o'clock at night.
			<br><br> <span class="speaker">TI:</span> And what would you be wearing, describe the whole routine for me. Like walk through taking a bath.
			<br><br><span class="speaker">LW: </span> Oh, (going to the) bathhouse... we just wear regular clothes and go up there. We didn't have no pajamas or anything like that to wear. It's right next door anyway. And when we go to the bathhouse, we stayed there about an hour. We do, we used to have more fun in the bathhouse than staying home.
			<br><br><span class="speaker">TI:</span> And who would you go with to the bathhouse?
			<br><br><span class="speaker">LW:</span> Well, it's the neighborhood, so you always run into some friends there. And next thing you notice, you'll be playing more than you're taking a bath.
			</div>	
		</div>
	</div>
	<br>
	<div class= "side-by-side">
		<div class="side">
			<div class="title">Fishing at Terminal Island</div>
			<div class="text">
				<span class="speaker">Kristen Luetkemeier:</span> You said your dad was a fisherman. Did he own a boat?<br><br>
				<span class="speaker">Minoru Shibata:</span> No. He joined the crew of one of the boats. There were basically two types of fishing boats, those that went out for the day, which came back in the evening, and those that went out further to Baja, Mexico, and north to the San Francisco area to fish. And those were the ones that were gone for maybe two or three weeks or a month, and my father used to be on one of those.<br><br>
				<span class="speaker">KL:</span> Yeah, I didn't realize that until I was, the first time I really talked to somebody from Terminal Island. I was surprised by how far the ships went. So he would be gone for a month or a couple weeks?<br><br>
				<span class="speaker">MS:</span> Roughly, right. It wasn't one of those boats that went out and came back the same day.<br><br>
				<span class="speaker">KL:</span> How was that for your family?<br><br>
				<span class="speaker">MS:</span> You know, I didn't have any reason to feel his absence. I was very comfortable, I think my sister felt the same way. I think it was because of the way the families were reared to behave. In other words, the siblings supported each other, and, of course, the mother took care, basically took care of the kids.
			</div>
			</div>
		<div class="side">
			<div class="title">Fishing at Manzanar Concentration Camp</div>
			<div class="text">
			<span class="speaker">Martha Nakagawa:</span> I want to ask you a little bit about fishing at Manzanar now. Who did you go fishing with and how did you manage to go fishing from a concentration camp?<br><br>
			<span class="speaker">Archie Miyatake:</span> [Laughs] Well, my cousin's husband, his name is Mike Nishita, and he was quite a fisherman and he loved trout fishing. He used to do that prior to World War II and he knew the trout fishing was very good in where we were… So one morning, one day he said, "I'm gonna go fishing this weekend, so if you want to go we're gonna have to start about two or three o'clock in the morning." I said sure, so what we did was we went to dig for worms…and we had to go while it was dark so we could crawl through the barbed wire and go, go out fishing… and we would hike and hike and go up the stream, and by the time we got to the foot of the mountain it was about five or six in the morning yet…we kept close to the creek in the area where we cannot be seen that much. So we fished and fished and we finally came back to the camp area. By that time we had quite a few trout...Mike had so many fish that he stuck it in his pocket and things like that because that was the only way he could carry it. Well, I only had about three or four trout. I wasn't that good… By the time we got back to the, near the camp it was still dark, so we were able to sneak back in. And then after we got back to apartment and got the fish out, and boy, my mother cooked the fish and it was so good. It was quite satisfying. 
		</div>
		</div>
	</div>
	<div class="img-container">
		<img src="src/data/dispersion.png" alt="dispersion plot of questions asked">
	</div>
</div>

<style>

.container {
	width: 100%;
	height: 100%;
	background-color: rgb(255, 255, 255);
}
.topic-sample {
	width: 33%;
	text-align: left;
	margin: auto;
	font-family: "minion-pro", "itc-avant-garde-gothic-pro","franklin-gothic-urw","adobe-garamond-pro", "Georgia",  Georgia, Helvetica, sans-serif;
	font-weight: 500;
	font-size: 1.05rem;
}

.img-container {
	width: 55%;
	margin: auto;
}
img {
	height: auto;
	width: auto;
}
.topic-sample .title {
	font-family: "itc-avant-garde-gothic-pro";
	font-size: 1rem;
	margin-bottom: 2%;
	color: lab(54 44 40);
	font-weight: 600;
}
.question {
	margin-bottom: 2%;
}
.side-by-side {
	display: flex;
	flex-direction: row;
	flex: 1 1 0px;
	gap: 5%;
	width: 60%;
	margin: auto;
}
.side {
	width: 50%;
}
.side .title {
	margin: auto;
	padding: 1.5%;
	padding-left: 2.5%;
	border-top-left-radius: 4px;
	border-top-right-radius: 4px;
	background-color: rgb(35, 29, 26);
	color: white;
    font-family: "itc-avant-garde-gothic-pro";
	font-size: .8rem;


}
.speaker {
	font-family: "itc-avant-garde-gothic-pro";
	font-weight: 700;
	font-size: .8rem;

}
.text {
	border: 1px solid rgb(35, 29, 26);
    font-family: "minion-pro", "itc-avant-garde-gothic-pro","franklin-gothic-urw","adobe-garamond-pro", "Georgia",  Georgia, Helvetica, sans-serif;
	font-weight: 300;
	padding: 2.5%;
	font-size: .95rem;
	border-bottom-left-radius: 4px;
	border-bottom-right-radius: 4px;
	max-height: 38.75svh;
	overflow-y: scroll;

}
</style>

