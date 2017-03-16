# What is Logic?

Logic enables you to extend your API with business logic hosted on your
own infrastructure. Different apps have different workflows and logic lets you customize Scaphold to fit virtually any need. You can host your logic on-prem or in the cloud as long as its accessible over the internet and you can even authenticate requests with custom headers. If your looking for the quickest way to start, we recommend
<a href="https://aws.amazon.com/lambda/" target="_blank">AWS Lambda</a>,
<a href="https://azure.microsoft.com/en-us/services/functions/" target="_blank">Azure Functions</a>, or
<a href="https://webtask.io/" target="_blank">Webtask.io</a>.
<hr />
Our logic system is based on the concept of
<a href="https://en.wikipedia.org/wiki/Function_composition_(computer_science)" target="_blank">function composition</a>.
Composition lets us combine small microservices to create extremely powerful workflows. Let's take a look at how they work.