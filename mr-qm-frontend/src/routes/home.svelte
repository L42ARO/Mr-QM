<script>
//IMPORTS
  import TopBar from "$components/TopBar.svelte";
  import { onMount } from "svelte";
  import "../theme/tailwind.css";
//GLOBAL VARIALBES
  var variables = 4;
  var minTermNum = 4;
  $:minterms=[];
  $:currSolution = '';
  var showSolution = false;
  var varsChosen = false;
  var mintermsChosen = false;

//REF VARIABLES
  var numVarInput;
  var numMintermsInput;
  var mtInput;

//FUNCTIONS
  $:varsUpdate = () => {
    variables = numVarInput.value;
    varsChosen = true;
    if(minTermNum > Math.pow(2, variables))
      minTermNum = 0;
      mintermsChosen = false;
    if(mintermsChosen)
      NewOne();
  };
  $:mintermsUpdate = () => {
    minTermNum = numMintermsInput.value;
    mintermsChosen = true;
    NewOne();
  };
  const NewOne = ()=>{
    showSolution = false;
    CreateMinterms(numVarInput.value);
  }
  const CreateMinterms = (variables) => {
    //generate max num using the number of variables as the maxnumber of bits
    let maxNum = Math.pow(2, variables)-1;
    minterms = [];
    //If the number of minterms is the max for the given number of variables, just return all minterms
    if (minTermNum == Math.pow(2, variables)) {
      for (let i = 0; i < Math.pow(2,variables); i++) {
        minterms.push(i);
      }
      return;
    }
    //Create random int minterms based on maxNum and minTermNum, (maxNum inclusive)
    for (let i = 0; i < minTermNum; i++) {
      //Make sure no duplicates
      let newMinterm = Math.floor(Math.random() * (maxNum+1));
      while (minterms.includes(newMinterm)) {
        newMinterm = Math.floor(Math.random() * maxNum);
      }
      //push
      minterms.push(newMinterm);
      //minterms = [6,9,7,0,8,10,14,11,5]
    }

  };
  //------ QUINE-MCCLUSKEY ALGORITHM ------
function quineMcCluskey(minterms) {
  var theoreticalMax = Math.pow(2, variables);
  if(minterms.length == 0)
    return '';
  if(minterms.length == theoreticalMax)
    return '1';

  //convert minterms to binary and pad with zeros according to the number of variables
  const binaryMinterms = minterms.map(m => m.toString(2).padStart(variables, '0'));
  
  // Create an array of groups, where each group contains minterms with the same number of 1s
  let groups = [];
  for (let i = 0; i < binaryMinterms.length; i++) {
    const numOnes = binaryMinterms[i].split('').filter(bit => bit === '1').length;
    if (!groups[numOnes]) {
      groups[numOnes] = [];
    }
    groups[numOnes].push(binaryMinterms[i]);
  }
  
  // Perform the Quine-McCluskey algorithm to find the minimal set of prime implicants
  let primeImplicants = [];
  let essentialPrimeImplicants = [];
  console.log(groups.length)
  while (groups.length > 0) {
    let alreadyUsed = [];
    const newGroups = [];
    for (let i = 0; i < groups.length; i++) {
      if(!groups[i]) continue;
      //If next group does not exist, add all terms in current group to prime implicants
      if(!groups[i+1]){
        for (let j = 0; j < groups[i].length; j++) {
          if(!alreadyUsed.includes(groups[i][j])){
            console.log(`Not found, adding; ${groups[i][j]}`)
            console.log(`Already used: ${alreadyUsed}`)
            primeImplicants.push(groups[i][j]);
            alreadyUsed.push(groups[i][j]);
          }
        }
        continue;
      }
      const group1 = groups[i];
      const group2 = groups[i + 1];
      const merged = [];
      for (let j = 0; j < group1.length; j++) {
        var currTermPrime = true;
        for (let k = 0; k < group2.length; k++) {
          const diffIndex = getDiffIndex(group1[j], group2[k]);
          if (diffIndex !== -1) {
            currTermPrime = false;
            const mergedTerm = group1[j].slice(0, diffIndex) + '-' + group1[j].slice(diffIndex + 1);
            alreadyUsed.push(group1[j]);
            alreadyUsed.push(group2[k]);
            console.log(`Found, merging; ${group1[j]} and ${group2[k]}`)
            if (!merged.includes(mergedTerm)) {
              merged.push(mergedTerm);
            }
          }
        }
        //If current term was not merged with any other term, add it to prime implicants
        if(currTermPrime){
          if(!alreadyUsed.includes(group1[j])){
            primeImplicants.push(group1[j]);
            alreadyUsed.push(group1[j]);
            console.log(`Not found, adding; ${group1[j]}`)
          }
        } 
      }
      newGroups.push(merged);
    }
    groups = newGroups;
    console.log(`Groups: ${groups}`)
    //primeImplicants = primeImplicants.concat(groups.flat());
  }
  console.log(`Prime implicants: ${primeImplicants}`)
  // Find the essential prime implicants and generate the minimized expression
  const coveredMinterms = new Set();
  let minimizedExpression = '';
  for (let i = 0; i < minterms.length; i++) {
    const binaryMinterm = binaryMinterms[i];
    let matchedPrimeImplicants = [];
    for (let j = 0; j < primeImplicants.length; j++) {
      if (coversMinterm(primeImplicants[j], binaryMinterm)) {
        matchedPrimeImplicants.push(primeImplicants[j]);
      }
    }
    if (matchedPrimeImplicants.length === 1) {
      essentialPrimeImplicants.push(matchedPrimeImplicants[0]);
      //Add all minterms covered by this prime implicant to the set of covered minterms
      for (let j = 0; j < binaryMinterms.length; j++) {
        if (coversMinterm(matchedPrimeImplicants[0], binaryMinterms[j])) {
          coveredMinterms.add(binaryMinterms[j]);
        }
      }
      //coveredMinterms.add(minterms[i]);
    }
  }
  //Get rid of duplicates in covered minterms
  const coveredMTs = [...new Set(coveredMinterms)];
  //Get rid of duplicates in essential primes
  essentialPrimeImplicants = [...new Set(essentialPrimeImplicants)];
  console.log(`Essential primes: ${essentialPrimeImplicants}`)
  console.log(`Covered minterms: ${coveredMTs}`);
  //Create list to hold varaints of non essential primes
  let nonEssentialPrimes = [];
  if(essentialPrimeImplicants.length<minterms.length){
    //Get list of remaining primes
    let remainingPrimes = primeImplicants.filter((prime) => {
      return !essentialPrimeImplicants.includes(prime);
    });
    console.log(`Remaining primes: ${remainingPrimes}`)
    //Go through remaining primes
    for(let i =0; i<remainingPrimes.length; i++){
      //Go through other remaining primes that are not itself
      for(let v=0; v<remainingPrimes.length; v++){
        if(v==i) continue;
        var coveredByI=0;
        var coveredByV=0;
        //Go through minterms
        for (let j = 0; j < binaryMinterms.length; j++) {
          if(coveredMTs.includes(binaryMinterms[j])) continue;
          if (coversMinterm(remainingPrimes[i], binaryMinterms[j])) {
            coveredByI++;
          }
          if (coversMinterm(remainingPrimes[v], binaryMinterms[j])) {
            coveredByV++;
          }
        }
        console.log(`Prime ${remainingPrimes[i]} covers ${coveredByI} minterms`)
        if(coveredByI == coveredByV || coveredByI > coveredByV){
          //If remainingPriems[i] covers any minterm already in nonEssentialPrimes, remove the covered minterm from nonEssentialPrimes
          for(let x=0; x<nonEssentialPrimes.length; x++){
            if(coversMinterm(remainingPrimes[i], nonEssentialPrimes[x])){
              console.log(`Removed ${nonEssentialPrimes[x]} from nonEssentialPrimes`)
              nonEssentialPrimes = nonEssentialPrimes.splice(x,1);
            }
          } 
          //Add prime i to list of non essential primes
          nonEssentialPrimes.push(remainingPrimes[i]);
          //Add all minterms covered by this prime implicant to the set of covered minterms if they are not already covered
          for (let j = 0; j < binaryMinterms.length; j++) {
            if (coversMinterm(remainingPrimes[i], binaryMinterms[j]) && !coveredMTs.includes(binaryMinterms[j])) {
              coveredMTs.push(binaryMinterms[j]);
            }
          }
        }   

        //if(coversMinterm(remainingPrimes[i], remainingPrimes[v])|| remainingPrimes[i]==remainingPrimes[v]){
        //  //Add prime i to list of non essential primes
        //  nonEssentialPrimes.push(remainingPrimes[i]);
        //}
      }
    }
  }
  //Get rid of duplicates
  nonEssentialPrimes = [...new Set(nonEssentialPrimes)];
  console.log(`Non essential primes: ${nonEssentialPrimes}`)
  var allPrimes = essentialPrimeImplicants.concat(nonEssentialPrimes);
  console.log(`All primes: ${allPrimes}`)
  //Get rid of duplicates
  allPrimes = [...new Set(allPrimes)];
  for (let i = 0; i < allPrimes.length; i++) {
    minimizedExpression += getExpressionFromPrimeImplicant(allPrimes[i]) + ' + ';
  }
  console.log('Essential prime implicants: ' + allPrimes);
  return minimizedExpression.slice(0, -3); // Remove trailing ' + '
}

// Helper function to get the index of the first differing bit in two binary strings
function getDiffIndex(binary1, binary2) {
  let diffIndex = -1;
  for (let i = 0; i < binary1.length; i++) {
    if (binary1[i] !== binary2[i]) {
      if (diffIndex !== -1) {
        return -1; // More than one differing
      }
      diffIndex = i;
    }
  }
  return diffIndex;
}
// Helper function to check if a prime implicant covers a minterm
function coversMinterm(primeImplicant, binaryMinterm) {
  for (let i = 0; i < binaryMinterm.length; i++) {
    if (primeImplicant[i] !== '-' && primeImplicant[i] !== binaryMinterm[i]) {
      return false;
    }
  }
  return true;
}

// Helper function to generate the boolean expression for a prime implicant
function getExpressionFromPrimeImplicant(primeImplicant) {
  let expression = '';
  for (let i = 0; i < primeImplicant.length; i++) {
    if (primeImplicant[i] === '0') {
      expression += String.fromCharCode(97 + i) + "'";
    } else if (primeImplicant[i] === '1') {
      expression += String.fromCharCode(97 + i);
    }
  }
  return expression;
}

//QM Algorithm
    //Convert minterms to binary
    //Create list for tableHistory
    //Create a dict for currGroups and nextGroups
    //Group minterms by number of 1's on currGroups (use 1's as key)
    //Create list for primeImplicants
    //ENTER BIG LOOP: while currGroups has groups to compare (more than 1)
      //Go through each group in currGroups except last one
        //Go through each minterm in group
          //Go through each minterm in next group
            //If minterms only differ by 1 bit
              //Check if either has an X
                //If X is in different position, skip
              //Add botth minterms to a new group inside nextGroups
          //If no new groups were created, add minterm(s) to primeImplicants
      //Add currGroups to tableHistory
      //currGroups now becomes nextGroups
    //IF currGroups isn't empty, add it to tableHistory
    //PRIME IMPLICANT TABLE
    //Create dict for primeImplicantTable (key = minterm, value = list of prime implicant groups)
    //Go through each original minterm
      //Go through each prime implicant group 
        //If prime implicant group covers minterm
          //Add prime implicant Group to list of current minterm
    //ESSENTIAL PRIME IMPLICANTS
    //Create list for essentialPrimeImplicants
    //Create list for mutable primeImplicantTable
    //BIG LOOP: while essential primes are still being found
      //Flag assume no more essential primes
      //Go through each minterm in mutablePrimeImplicantTable
        //If minterm has only 1 prime implicant group
          //Flag that an essential prime was found
          //Add prime implicant group to essentialPrimeImplicants
          //Remove all minterms in prime implicant group from mutablePrimeImplicantTable
          //Remove prime implicant group from all other minterms in mutablePrimeImplicantTable
    //IF not all minterms were covered by essential primes
      //Create reaminingPrimeImplicants, add all prime implicants in mutablePrimeImplicantTable
      //Go through remining primeImplicants with i
        //Go through remining primeImplicants with j
          //If i and j are different
            //If i covers j
              //Remove j from remainingPrimeImplicants
            //If j covers i
              //Remove i from remainingPrimeImplicants
            //If i and j are the same
              //Remove j from remainingPrimeImplicants
      //Add remainingPrimeImplicants to essentialPrimeImplicants
    //SOLUTION
    //Create solution string
    //Go through each essential prime implicant
      //Add prime implicant to solution string (translate 0001 and 0x02 to letters) //Add OR to solution string

  $:GetMintermsText = () => {
    let mintermsText = "";
    minterms.forEach((minterm) => {
      mintermsText += minterm + ",";
    });
    //Crop trailing comma
    mintermsText = mintermsText.slice(0, -1);
    return mintermsText;
  };
  
</script>

<!--<TopBar useMenu = {false}/>-->
<ion-content id="router-content">
  <!-- Div that acts like a card-->
  <div class="w-full h-full bg-gradient-to-r from-indigo-400 to-white dark:from-indigo-900 dark:to-black flex flex-col justify-start items-center">
    <div class="text-4xl py-3 m-3 px-16 rounded-3xl font-bold font-mono bg-gradient-to-r from-blue-400 to-green-400 dark:from-blue-600 dark:to-green-600">Mr QM</div>
    <div class="w-90 h-90 card p-8 flex flex-col items-center">
      <div class="font-mono font-bold">Do you want a QM exercise?</div>
      <br/>
      <div class="font-mono">How many variables? (A, B, C, ...)</div>
      <ion-segment bind:this={numVarInput} on:ionChange={()=>varsUpdate()}>
        {#each {length:6} as _, i}
          <ion-segment-button value={i+1}>{i+1}</ion-segment-button>
        {/each}
      </ion-segment>

      {#if varsChosen}
      <br />
      <div class="font-mono">How many minterms?</div>
      <div class="w-16 px-2 rounded-lg bg-gray-200 dark:bg-gray-800">
      <ion-input bind:this={numMintermsInput} on:change={()=>mintermsUpdate()} type="number" min="1" max={Math.pow(2,variables)}></ion-input>
      </div>
      {#if mintermsChosen}
      <br/>
      <ion-label> Minimize the following ∑m({GetMintermsText()})</ion-label>
      <br/>
      {#if !showSolution}
      <ion-button on:click={()=>{
        currSolution = quineMcCluskey(minterms);
        showSolution = true;
      }}>Show Solution</ion-button>
      {/if}
      {#if showSolution}
        <ion-label>Answer</ion-label>
        <ion-label class="bg-gray-200 dark:bg-gray-800 rounded-2xl p-2 m-2"> {currSolution} </ion-label>
      {/if}
      <ion-button on:click={()=>NewOne()}>Another!</ion-button>
      {/if}
      {/if}
    </div>
    
  </div>
</ion-content>
