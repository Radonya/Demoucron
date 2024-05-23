<template>
  <div class="w-full ">
    <div class="w-full bg-gray-200  flex justify-evenly shadow-lg rounded-xl">
      <div class="flex flex-rox w-1/2  justify-center ">
          <form @submit.prevent="ajouterNoeud"
            class="shadow-lg rounded-lg w-full flex item-center">
            <div class="w-full">
              <div class="flex flex-row items-center m-4 ">
                <div class="m-4 flex flex-row">
                 
                  <input v-model="newNodeLabel" type="text" placeholder="Sommet de départ"
                    class="p-2 border border-gray-200 rounded " />
                </div>
                
                <div class="m-4 flex w-full flex-row justify-between">
                  <button type="submit" class="px-4 py-2 w-32 bg-blue-500 text-white rounded  hover:bg-blue-700">
                    Ajouter
                  </button>
                  <button type="button" @click="supprimerNoeud" :disabled="!selectedNode"
                    class="px-4 py-2 w-32 bg-red-500 text-white rounded  hover:bg-red-700">
                    Supprimer
                  </button>
                  <button @click="runAlgo(1)" class="px-2 py-2 w-32 bg-amber-500 text-white rounded hover:bg-amber-800">
              Min
            </button>
            <button @click="runAlgo(0)" class="px-2 py-2 w-32 bg-gray-500  hover:bg-gray-800 text-white rounded">
              Max
            </button>
                </div>

              </div>
            </div>
          </form>
        </div>
    </div>
    <div class="m-12">
      <div class="flex flex-row justify-between m-8 ">
        
        <div class="flex">
          <div class="w-[70vw] h-[60vh] bg-white  rounded-lg border-2 " ref="graphContainer"></div>
        </div>
        <div v-if="optimalPath.length > 0" class="flex w-1/2 flex-col shadow-xl rounded-xl shadow-green-100 items-center ">
        <div class="bg-white w-[50%]  p-4 flex flex-col items-center">
          <div class="flex w-full flex-row p-4 justify-evenly">
            <div class="flex flex-col justify-between items-center">
              <span>Début</span>
              <span class="flex items-center justify-center h-8 w-8 rounded-full bg-cyan-500 text-white">
                {{ startNodeGraph }}
              </span>
            </div>
            <div class="flex flex-col justify-between  items-center">
              <span>Fin</span>
              <span class="flex items-center justify-center h-8 w-8 rounded-full bg-red-500 text-white">
                {{ endNodeGraph }}
              </span>
            </div>
          </div>

        </div>
        <div v-if="optimalPath.length > 0" class="flex flex-col w-[50%]  items-center mb-8 ">
          <h2 class="text-xl font-bold mb-2 flex justify-center">Chemins</h2>
          <div class="bg-white w-full flex flex-row justify-between items-center">
            <ul>
              <li v-for="(path, index) in optimalPath" :key="index" class="flex justify-center items-center text-xl ">
                {{ path.join("->") }}
              </li>
            </ul>
          </div>
        </div>
        <div v-else>
          <p>Aucun chemin optimal trouvé.</p>
        </div>

      </div>
      </div>

      
      <div v-if="adjacencyMatrix.length > 0" class="flex flex-row items-center justify-center space-x-12  w-full mt-12  ">
        <div class=" flex ">
          <div class="bg-white shadow-lg rounded-lg p-4">
            <h2 class="text-xl font-bold mb-2">Matrice</h2>
            <table class="border-collapse border border-gray-300 w-full">
              <thead>
                <tr>
                  <th></th>
                  <th v-for="vertex in adjacencyMatrixVertices" :key="vertex"
                    class="border border-gray-300 p-2 text-center">
                    {{ vertex }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, rowIndex) in adjacencyMatrix" :key="rowIndex">
                  <td class="border border-gray-300 p-2 text-center">{{ adjacencyMatrixVertices[rowIndex] }}</td>
                  <td v-for="(value, colIndex) in row" :key="colIndex" class="border border-gray-300 p-2 text-center">
                    {{ value === Infinity ? '∞' : value }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <div v-if="lastMatrix.length" class="flex">
          <div class="bg-white shadow-lg rounded-lg p-4 ">
            <h2 class="text-xl font-bold mb-2">Matrice Finale</h2>
            <table class="border-collapse border border-gray-300 w-full">
              <thead>
                <tr>
                  <th></th>
                  <th v-for="vertex in adjacencyMatrixVertices" :key="vertex"
                    class="border border-gray-300 p-2 text-center">
                    {{ vertex }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, rowIndex) in lastMatrix" :key="rowIndex">
                  <td class="border border-gray-300 p-2 text-center">{{ adjacencyMatrixVertices[rowIndex] }}</td>
                  <td v-for="(value, colIndex) in row" :key="colIndex" class="border border-gray-300 p-2 text-center"
                    >
                    {{ value === Infinity ? '∞' : value }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>



      </div>
    </div>
  </div>
</template>

<script>
import { DataSet, Network } from "vis-network/standalone";

export default {
  data() {
    return {
      startNode: "",
      endNode: "",
      nodes: new DataSet(),
      edges: new DataSet(),
      edgeValueInput: "",
      selectedNode: null,
      arcValues: [{}],
      startNodeGraph: "",
      endNodeGraph: "",
      nouveauLabel: "",
      fordResult: null,
      adjacencyMatrix: [],
      adjacencyMatrixVertices: [],
      matricesDataProperty: [],
      nodeLabels: [],
      initialMatrix: [],
      lastMatrix: [],
      lastColumnMin: [],
      lastColumnMin: [],
      minNodeIds: [],
      minEdgeIds: [],
      optimalPath: [],
      newNodeLabel:""
    };
  },

  mounted() {
    this.renderGraph();
  },

  methods: {
    renderGraph() {
      const data = {
        nodes: this.nodes,
        edges: this.edges,
      };
      const options = {
        interaction: {
          selectConnectedEdges: false,
        },
        edges: {
          arrows: {
            to: {
              enabled: true,
              scaleFactor: 0.5,
              type: "arrow",
            },
          },
          color: {
            inherit: false,
          },
        },
      };

      const container = this.$refs.graphContainer;

      const network = new Network(container, data, options);

      network.on("click", (event) => {
        const { nodes, edges } = event;
        if (edges.length === 1) {
          const edgeId = edges[0];
          const edge = this.edges.get(edgeId);
          const newEdgeValue = prompt("Entrez la valeur de l'arc :");
          if (newEdgeValue !== null) {
            edge.label = newEdgeValue.trim();
            this.edges.update(edge);
          }
        } else if (nodes.length > 0) {
          const clickedNodeId = nodes[0];
          if (this.selectedNode && this.selectedNode !== clickedNodeId) {
            const newEdge = {
              from: this.selectedNode,
              to: clickedNodeId,
              label: this.edgeValueInput.trim(),
            };
            this.edges.add(newEdge);
            this.selectedNode = null;
            this.edgeValueInput = "";
            this.renderGraph();
          } else {
            this.selectedNode = clickedNodeId;
          }
        }
      });
    },

    ajouterNoeud() {
      if (this.newNodeLabel.trim() === "") {
        return;
      }

      let id = this.generateUniqueId();
      while (this.nodes.get(id)) {
        id = this.generateUniqueId();
      }

      const newNode = {
        id: id,
        label: this.newNodeLabel.trim(),
        color: "#A5B4FC",
      };

      this.nodes.add(newNode);
      this.newNodeLabel=""
      this.renderGraph();
    },

    generateUniqueId() {
      let id = this.nodes.length + 1;
      while (this.nodes.get(id)) {
        id++;
      }
      return id;
    },

    supprimerNoeud() {
      if (this.selectedNode === this.startNode) {
        this.startNode = "";
      } else if (this.selectedNode === this.endNode) {
        this.endNode = "";
      }
      this.nodes.remove(this.selectedNode);
      this.edges.remove(
        this.edges.get({
          filter: (edge) => edge.from === this.selectedNode || edge.to === this.selectedNode,
        })
      );
      this.selectedNode = null;
    },

    // Ajoutez une méthode pour trouver le chemin maximal
    findMaximalPath(resultMatrix) {
      const numVertices = resultMatrix.length;
      const vertices = this.adjacencyMatrixVertices;
      let maxPath = [];
      let maxWeight = -Infinity;

      // Parcourez chaque sommet pour trouver le chemin maximal
      for (let i = 0; i < numVertices; i++) {
        for (let j = 0; j < numVertices; j++) {
          // Vérifiez si le poids du chemin est maximal et valide
          if (resultMatrix[i][j] !== Infinity && resultMatrix[i][j] > maxWeight) {
            maxWeight = resultMatrix[i][j];
            maxPath = this.constructPath(vertices[i], vertices[j], resultMatrix);
          }
        }
      }
      return { path: maxPath, weight: maxWeight };
    },

    // Ajoutez une méthode pour construire le chemin maximal à partir de la matrice de résultats
    constructPath(startNode, endNode, resultMatrix) {
      const path = [startNode];
      let currentNode = startNode;

      // Continuez à ajouter des nœuds jusqu'à atteindre le nœud final
      while (currentNode !== endNode) {
        for (let i = 0; i < resultMatrix.length; i++) {
          // Recherchez le prochain nœud dans la matrice de résultats
          if (resultMatrix[this.adjacencyMatrixVertices.indexOf(currentNode)][i] === resultMatrix[this.adjacencyMatrixVertices.indexOf(currentNode)][this.adjacencyMatrixVertices.indexOf(endNode)] - resultMatrix[this.adjacencyMatrixVertices.indexOf(endNode)][i]) {
            path.push(this.adjacencyMatrixVertices[i]);
            currentNode = this.adjacencyMatrixVertices[i];
            break;
          }
        }
      }
      return path;
    },
    // Appelez cette méthode dans votre méthode runAlgo()
    runAlgo(type) {
      const adjacencyMatrix = this.graphToMatrix();
      this.initialMatrix = this.deepCopy(adjacencyMatrix);
      if (type) {

        this.fordResult = this.demoucronAlgorithm(adjacencyMatrix);
        this.optimalPath = this.findOptimalPaths(this.fordResult, this.adjacencyMatrixVertices, this.initialMatrix);
        if (this.optimalPath.length > 0) {
          this.coloration(this.optimalPath)
        }

        //   const dijkstraResult = this.dijkstraAlgorithm(this.fordResult, this.adjacencyMatrixVertices[0]);

      } else {
        this.fordResult = this.demoucronMax(adjacencyMatrix);
        this.optimalPath = this.findMaximalPaths(this.fordResult, this.adjacencyMatrixVertices, this.initialMatrix); // Appel pour trouver le chemin maximal
        if (this.optimalPath.length > 0) {
          this.coloration(this.optimalPath)
        }

      }
    },
    findOptimalPaths(resultMatrix, vertices, initialMatrix) {
      const numVertices = resultMatrix.length;
      const optimalPaths = [];
      const totalWeight = resultMatrix[0][numVertices - 1];
      console.log(totalWeight)
      function findPaths(currentIndex, path, visited, currentWeight) {
        path.push(vertices[currentIndex]);
        visited[currentIndex] = true;
        if (currentIndex === 0 && currentWeight === totalWeight) {
          optimalPaths.push([...path].reverse());
        } else {
          for (let prevIndex = numVertices - 1; prevIndex >= 0; prevIndex--) {
            if (!visited[prevIndex] && initialMatrix[prevIndex][currentIndex] !== Infinity) {
              const weight = initialMatrix[prevIndex][currentIndex];
              findPaths(prevIndex, path, visited.slice(), currentWeight + weight);
            }
          }
        }

        path.pop();
      }
      findPaths(numVertices - 1, [], new Array(numVertices).fill(false), 0);
      console.log(optimalPaths)
      return optimalPaths;
    }


    , findMaximalPaths(resultMatrix, vertices, initialMatrix) {
      const numVertices = resultMatrix.length;
      const maximalPaths = [];
      const totalWeight = resultMatrix[0][numVertices - 1];

      function findPaths(currentIndex, path, visited, currentWeight) {
        path.push(vertices[currentIndex]);
        visited[currentIndex] = true;

        if (currentIndex === 0 && currentWeight === totalWeight) {
          maximalPaths.push([...path].reverse());
        } else {
          for (let prevIndex = numVertices - 1; prevIndex >= 0; prevIndex--) {
            if (!visited[prevIndex] && initialMatrix[prevIndex][currentIndex] !== Infinity) {
              const weight = initialMatrix[prevIndex][currentIndex];
              findPaths(prevIndex, path, visited.slice(), currentWeight + weight);
            }
          }
        }

        path.pop();
      }

      findPaths(numVertices - 1, [], new Array(numVertices).fill(false), 0);
      console.log(maximalPaths)
      return maximalPaths;
    },


    deepCopy(matrix) {
      return matrix.map(row => [...row]);
    },
    graphToMatrix() {
      const nodes = this.nodes.get();
      const edges = this.edges.get();
      const nodesWithOutgoingEdges = new Set(edges.map(edge => edge.from));
      nodes.sort((a, b) => a.label.localeCompare(b.label));
      const sortedNodes = [
        ...nodes.filter(node => nodesWithOutgoingEdges.has(node.id)),
        ...nodes.filter(node => !nodesWithOutgoingEdges.has(node.id))
      ];

      const matrix = [];
      const vertices = [];
      for (const node of sortedNodes) {
        vertices.push(node.label);
        matrix.push(Array(sortedNodes.length).fill(Infinity));
      }

      for (const edge of edges) {
        const fromIndex = sortedNodes.findIndex(node => node.id === edge.from);
        const toIndex = sortedNodes.findIndex(node => node.id === edge.to);
        const weight = parseInt(edge.label);

        matrix[fromIndex][toIndex] = weight;
      }
      this.startNodeGraph = vertices[0];
      this.endNodeGraph = vertices[vertices.length - 1];
      this.adjacencyMatrix = matrix;
      this.adjacencyMatrixVertices = vertices;
      return matrix;
    }

    ,

    demoucronAlgorithm(adjMatrix) {
      const n = adjMatrix.length;
      let dkMatrix = this.deepCopy(adjMatrix);
      const matricesData = [];

      for (let k = 1; k < n - 1; k++) {
        for (let i = 0; i < n; i++) {
          for (let j = 0; j < n; j++) {
            if (i !== j && dkMatrix[i][k] !== Infinity && dkMatrix[k][j] !== Infinity) {
              const wijKMinus1 = dkMatrix[i][k] + dkMatrix[k][j];
              dkMatrix[i][j] = isFinite(dkMatrix[i][j]) ? Math.min(wijKMinus1, dkMatrix[i][j]) : wijKMinus1;
            }
          }
        }
        matricesData.push(this.deepCopy(dkMatrix));
      }
      this.updateMatrices(matricesData, "min");
      return dkMatrix;
    },

    demoucronMax(adjMatrix) {
      const n = adjMatrix.length;
      let dkMatrix = this.deepCopy(adjMatrix);
      const matricesData = [];
      for (let k = 1; k < n - 1; k++) {
        for (let i = 0; i < n; i++) {
          for (let j = 0; j < n; j++) {
            if (i !== j && dkMatrix[i][k] !== Infinity && dkMatrix[k][j] !== Infinity) {
              const wijKMinus1 = dkMatrix[i][k] + dkMatrix[k][j];
              dkMatrix[i][j] = isFinite(dkMatrix[i][j]) ? Math.max(wijKMinus1, dkMatrix[i][j]) : wijKMinus1;
            }
          }
        }
        matricesData.push(this.deepCopy(dkMatrix));
      }
      this.updateMatrices(matricesData, "max");
      return dkMatrix;
    },


    updateMatrices(matrices, arg) {
      this.matricesDataProperty = matrices;
      this.getLastMatrix(arg);
    },

    getLastMatrix(arg) {
      this.lastMatrix = this.matricesDataProperty[this.matricesDataProperty.length - 1];
      
    }
    ,
    coloration(path) {
  const visitedNodes = new Set();
  const visitedEdges = new Set();

  // Parcours de chaque chemin optimal
  path.forEach(subPath => {
    // Ajout de tous les nœuds visités dans un ensemble
    subPath.forEach(node => visitedNodes.add(node));

    // Marquage des arêtes entre les nœuds visités
    for (let i = 0; i < subPath.length - 1; i++) {
        const fromNode = subPath[i];
        const toNode = subPath[i + 1];

        // Recherche des arêtes correspondant aux nœuds connectés
        this.edges.forEach(edge => {
            const fromNodeId = this.nodes.getIds({ filter: node => node.label === fromNode })[0];
            const toNodeId = this.nodes.getIds({ filter: node => node.label === toNode })[0];
            if ((edge.from === fromNodeId && edge.to === toNodeId) || (edge.from === toNodeId && edge.to === fromNodeId)) {
                visitedEdges.add(edge.id);
            }
        });
    }
  });

  // Mise à jour de la couleur des nœuds visités en jaune
  visitedNodes.forEach(nodeLabel => {
    const nodes = this.nodes.get({
      filter: n => n.label === nodeLabel
    });
    nodes.forEach(node => {
      console.log(`Coloring node ${nodeLabel} yellow`);
      node.color = 'yellow';
      this.nodes.update(node);
    });
  });

  // Mise à jour de la couleur des arêtes visitées en rouge
  visitedEdges.forEach(edgeId => {
    const edge = this.edges.get(edgeId);
    if (edge) {
      console.log(`Coloring edge ${edgeId} red`);
      edge.color = { color: 'green' };
      this.edges.update(edge);
    } else {
      console.log(`Edge ${edgeId} not found`);
    }
  });

  // Coloration des nœuds non visités en gris
  this.nodes.forEach(node => {
    if (!visitedNodes.has(node.label)) {
      console.log(`Coloring node ${node.label} gray`);
      node.color = "#A5B4FC";
      this.nodes.update(node);
    }
  });

  // Coloration des arêtes non visitées en gris
  this.edges.forEach(edge => {
    if (!visitedEdges.has(edge.id)) {
      console.log(`Coloring edge ${edge.id} gray`);
      edge.color = { color: "#A5B4FC" };
      this.edges.update(edge);
    }
  });

  // Forcer le rendu du graphe après les mises à jour
  this.renderGraph();
}

  },
};
</script>

<style>
/* ... Styles CSS ... */
</style>
