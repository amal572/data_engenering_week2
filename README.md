<div>
<img src="https://github.com/mage-ai/assets/blob/main/mascots/mascots-shorter.jpeg?raw=true">
</div>

## Configuring Mage

Mage is an open-source, hybrid framework for transforming and integrating data. ✨

Before getting into some of the key components of Mage, we need to set up our environment. If you haven't already, you'll need to clone the [here](https://docs.mage.ai/introduction/overview) provided so we can set up our environment locally.

Clone repo to vs code
1. Open a terminal in your local repo directory
2. Run the following commands
3. Now, let's build the container
  ```bash
docker compose build
```

4. start the Docker container:
```bash
docker compose up
```
5. Navigate to http://localhost:6789 in your browser

Now we can start building our pipeline!

## Building an ETL Pipeline with Mage (PostgreSQL)

The first pipeline is an ETL pipeline using blocks to load the yellow taxi data from this GitHub repo, transform the data, and export the data into a Postgres database within our docker container. The resulting DAG should look something like the screenshot below.

<div>
<img src="https://github.com/amal572/data_engenering_week2/blob/main/image/week2.PNG">
</div>



## Assistance

1. [Mage Docs](https://docs.mage.ai/introduction/overview): a good place to understand Mage functionality or concepts.
2. [Mage Slack](https://www.mage.ai/chat): a good place to ask questions or get help from the Mage team.
3. [DTC Zoomcamp](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/week_2_workflow_orchestration): a good place to get help from the community on course-specific inquireies.
4. [Mage GitHub](https://github.com/mage-ai/mage-ai): a good place to open issues or feature requests.
