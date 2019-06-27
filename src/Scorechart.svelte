<script>
  export let players;

  const colors = [
    "#e74c3cbb",
    "#3498dbbb",
    "#2ecc71bb",
    "#f1c40fbb",
    "#9b59b6bb",
    "#e67e22bb"
  ];
  let chart;

  setTimeout(loadChart, 500);

  $: if (players) {
    updateChartData();
  }

  function loadChart() {
    chart = new Chart("myChart", {
      labels: [1],
      type: "line",
      data: {
        datasets: []
      },
      options: {
        scales: {
          xAxes: [
            {
              display: true,
              scaleLabel: {
                display: true,
                labelString: "Turn"
              }
            }
          ],
          yAxes: [
            {
              display: true,
              scaleLabel: {
                display: true,
                labelString: "Score"
              },
              ticks: {
                beginAtZero: true
              }
            }
          ]
        }
      }
    });
  }

  function updateChartData() {
    if (chart) {
      let chartData = [];
      let labels = [0];
      let pushLabels = true;
      players.forEach((player, pindex) => {
        let cuml_scs = [0];
        let cuml = 0;
        player.scores.forEach((score, i) => {
          cuml += score;
          cuml_scs.push(cuml);
          if (pushLabels) {
            labels.push(i + 1);
          }
        });

        pushLabels = false;

        const dataset = {
          label: player.name,
          data: cuml_scs,
          // backgroundColor: colors[pindex % colors.length],
          borderColor: colors[pindex % colors.length],
          fill: false
        };
        chartData.push(dataset);
      });

      chart.data.labels = labels;
      chart.data.datasets = chartData;

      chart.update();
    }
  }
</script>

<div>
  <canvas id="myChart" width="400" height="300" />
</div>
