<script lang="ts">
  import type * as cjs from 'chart.js';
  import People from '$lib/component/page/People.svelte';

  // Hand image
  import FromOnline from "../images/hands/from-online.png"

  // Statistics
  import { Line } from 'svelte-chartjs';
  import 'chart.js/auto';
  import results from '../stats/results.csv';
  import Landing from '$lib/component/page/Landing.svelte';

  const parsedResults = results.map((result) =>
    Object.fromEntries(Object.entries(result).map(([key, value]) => [key.trim(), value.trim()]))
  );

  const labels = parsedResults.map((result) => result.epoch);

  const dataFilters = ['epoch', 'lr/'];

  interface Category {
    name: string;
    labels: string[];
    checked: boolean;
    checkbox?: HTMLInputElement;
  }

  const categories: Category[] = [
    {
      name: 'Loss',
      labels: ['box_loss', 'cls_loss', 'dfl_loss'],
      checked: true
    },
    {
      name: 'Precision & Recall',
      labels: ['precision', 'recall', 'mAP50', 'mAP50-95'],
      checked: true
    }
  ];

  const rawData = parsedResults.reduce((acc: Record<string, string[]>, result) => {
    Object.entries(result).forEach(([key, value]) => {
      if (!dataFilters.some((filter) => key.includes(filter))) {
        if (!acc[key]) {
          acc[key] = [];
        }
        acc[key].push(value);
      }
    });
    return acc;
  }, {});

  const defaultOptions: Partial<cjs.ChartDataset<'line', number[]>> = {
    borderDash: [],
    borderDashOffset: 0.0,
    pointRadius: 0,
    pointHitRadius: 10,
    tension: 0.5
  };

  const datasets = Object.entries(rawData).map(([key, values]) => ({
    label: key,
    ...defaultOptions,
    data: values.map((value) => parseFloat(value))
  }));

  let data: cjs.ChartData<'line', number[], unknown> = {
    labels,
    datasets
  };
</script>

<svelte:head>
  <title>ASL Caption</title>
</svelte:head>

<main>
  <Landing />
  <div class="slant">
    <h2 id="about">About Us</h2>

    <div class="about-container">
      <div>
        <p>
          We're a group of students who are <b>passionate</b> about promoting inclusive communication
          for all. We firmly believe that <b>communication</b> is a fundamental human right, and helping
          people to communicate is our way of giving back to the community.
        </p>

        <p>
          Our mission is to create a world where everyone can <b>effectively</b> and <b>effortlessly</b>
          communicate, regardless of their hearing abilities. By recognizing the importance of American Sign
          Language and its role in facilitating communication for the deaf community, we have taken our first
          step towards making the world a more accessible place.
        </p>

        <p>
          We aim to take the first steps in ASL recognition by creating a model that can recognize the
          fundamental building blocks of language: the alphabet. We hope that our model can be used as a
          stepping stone for future research in ASL recognition, and that it can be used to help the deaf
          community communicate more effectively with their devices and others.
        </p>
      </div>

      <img src={FromOnline} alt="Labeled hand doing a 'X' pose" height="600" />
    </div>

    <h3 style="font-size: 35px">Our Team</h3>

    <p>Hover over the images to learn more about us!</p>

    <People />
  </div>
  <div class="color-container">
    <div class="slant slant-secondary">
      <h2>Evaluation</h2>

      <h3>
        We ran our model on <a
          style="color: var(--text-bold)"
          href="https://universe.roboflow.com/meredith-lo-pmqx7/asl-project">our 26-class ASL dataset</a
        > with <br/>100 epochs at resolution 640x640 with a pretrained <a href="https://github.com/ultralytics/ultralytics">yolov8n</a> base model,
        achieving mAP50 scores &gt; 95%.
      </h3>
      <p>(You can filter between loss or accuracy by clicking on the category buttons.)</p>

      <div id="evaluationGrid">
        <div class="graphContainer">
          <Line
            {data}
            options={{
              responsive: true,
              scales: {
                x: {
                  title: {
                    display: true,
                    text: 'Epoch',
                    color: 'white'
                  },
                  ticks: {
                    color: 'white'
                  },
                  grid: {
                    color: 'rgba(224, 224, 224, 0.2)'
                  }
                },
                y: {
                  ticks: {
                    color: 'white',
                    major: {
                      enabled: true
                    }
                  },
                  grid: {
                    color: 'rgba(224, 224, 224, 0.2)'
                  }
                }
              },
              color: 'white'
            }}
          />
          {#if categories.every((category) => !category.checked)}
            <div class="graphOverlay">
              <h2>There are currently no datasets enabled. Make sure to check one!</h2>
            </div>
          {/if}
        </div>

        {#each categories as category}
          <button
            class="categorySwitch {category.checked ? 'checked' : ''}"
            on:click={() => category.checkbox?.click()}
          >
            <input
              type="checkbox"
              bind:this={category.checkbox}
              bind:checked={category.checked}
              on:change={() => {
                for (const datasetName of category.labels) {
                  const dataset = datasets.find((dataset) => dataset.label === datasetName);

                  if (dataset) {
                    dataset.hidden = !category.checked;
                  }
                }

                data = data;
              }}
            />
            {category.name}
          </button>
        {/each}
      </div>
    </div>
  </div>
</main>

<footer>
  Source on <a href="https://github.com/LeoDog896/asl-caption">GitHub</a>
</footer>

<style>
  /* text to the left, image to the right */
  .about-container {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    max-width: 1200px;
    margin: 0 auto;
  }

  .about-container > div {
    width: 70%;
    padding: 0 2rem;
  }

  @media (max-width: 900px) {
    .about-container {
      flex-direction: column;
    }

    .about-container > div {
      width: 100%;
    }
  }

  main {
    margin-top: 2rem;
  }

  :root {
    --max-width: 1500px;
  }

  .categorySwitch {
    display: inline-block;
    cursor: pointer;
    font-weight: 800;
    padding: 0.5rem;
    color: white;
    border: 2px solid var(--secondary-hover);
    margin: 1rem;
    background-color: var(--secondary);
    transition: background-color 0.1s cubic-bezier(0.075, 0.82, 0.165, 1);
  }

  .categorySwitch:hover,
  .categorySwitch.checked {
    background-color: var(--secondary-hover);
  }

  footer {
    text-align: center;
    padding: 1rem;
    color: var(--background-text);
  }

  :global(.spin) {
    animation: spin 1s linear infinite;
  }

  p {
    max-width: var(--max-width);
    margin: 2rem auto;
  }

  #evaluationGrid {
    max-width: var(--max-width);
    margin: 0 auto;
  }

  .graphContainer {
    position: relative;
    margin: 0 auto;
    max-width: 1500px;
    display: flex;
    justify-content: center;
  }

  .graphOverlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 4rem;
    background-color: rgba(0, 0, 0, 0.5);
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }

    100% {
      transform: rotate(360deg);
    }
  }

  .slant {
    padding: 2rem;
    padding-top: 4rem;
    background-color: var(--primary);
    height: 100%;
    clip-path: polygon(0 5rem, 100% 0, 100% calc(100% + 5rem), 0 100%);
    text-align: center;
  }

  .slant-secondary {
    background-color: #00b3b3;
    clip-path: polygon(0 0, 100% 5rem, 100% 100%, 0 100%);
  }

  .color-container {
    background-color: var(--primary);
  }

  .slant p {
    font-size: 1.2rem;
    font-weight: 400;
    line-height: 1.5;
  }

  h2 {
    font-size: 2.7rem;
    text-align: center;
    font-weight: 800;
    line-height: 1.2;
    text-transform: uppercase;
    font-family: 'Outfit Variable', sans-serif;
  }
</style>
