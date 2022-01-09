<script>
  import { xml2js } from "xml-js";

  const keywords = [
    "corona",
    "omikron",
    "inzidenz",
    "luca-app",
    "impfung",
    "impfpflicht"
  ];

  let selectedSource = "Tagesschau";
  let articles = [];

  const feeds = {
    Tagesschau: "https://www.tagesschau.de/xml/rss2/",
    "SWR Aktuell": "https://www.swr.de/~rss/swraktuell/swraktuell-100.xml",
    "SWR Aktuell BW": "https://www.swr.de/~rss/swraktuell/swraktuell-bw-100.xml",
    "SWR Aktuell RP": "https://www.swr.de/~rss/swraktuell/swraktuell-rp-100.xml"
  };

  async function getFeed(name) {
    selectedSource = name;
    const resp = await fetch(feeds[name]);
    const text = await resp.text();
    const data = xml2js(text, { compact: true });
    articles = data.rss.channel.item
      .filter(item =>
        keywords.every(
          keyword =>
            !item.title._text.toLowerCase().includes(keyword) &&
            !item.description._text.toLowerCase().includes(keyword)
        )
      )
      .map(item => ({
        title: item.title._text,
        description: item.description._text,
        link: item.link._text,
        date: new Date(item.pubDate._text)
      }))
      .sort((a, b) => b.date.getTime() - a.date.getTime());
  }
  getFeed(selectedSource);
</script>

<style>
  :global(body) {
    background: #fafafa;
    font-family: Helvetica;
  }

  main {
    max-width: 600px;
    margin-left: auto;
    margin-right: auto;
  }

  nav {
    background: #333;
  }

  nav > ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  nav > ul > li {
    display: inline-block;
  }

  nav > ul > li.active,
  nav > ul > li:hover {
    background: #555;
  }

  nav > ul > li > button {
    display: inline-block;
    padding: 10px;
    color: #fff;
    text-decoration: none;
    background: transparent;
    border: none;
    cursor: pointer;
  }

  article {
    margin-top: 40px;
  }

  h2 {
    margin-bottom: 5px;
  }

  h2 > a {
    text-decoration: none;
    color: #111;
  }

  h2 > a:hover {
    text-decoration: underline;
  }

  h3 {
    margin-top: 0;
    color: #444;
  }
</style>

<main>
  <h1>{selectedSource} ohne Corona</h1>
  <nav>
    <ul>
      {#each Object.keys(feeds) as source}
      <li class={source === selectedSource ? "active" : ""}><button on:click={() => getFeed(source)}>{source}</button></li>
      {/each}
    </ul>
  </nav>
  {#each articles as article}
  <article>
    <h2><a href={article.link} target="_blank">{article.title}</a></h2>
    <h3>{article.date.toLocaleString()}</h3>
    <p>{article.description}</p>
  </article>
  {/each}
  <hr>
  <p>Gefilterte Keywords: {keywords.join(", ")}</p>
</main>
