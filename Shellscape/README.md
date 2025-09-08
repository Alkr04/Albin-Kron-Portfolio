# Shellscape
Enemy Programer

Duration: 11/2024 - 01/2025

Engine: Unity

Genre: First Person Shoter

Team: 3 Programers, 3 Artists

# My contribution
I was primaraly in charge of the enemy

the entire project was plagued by problems. which resulted in a lot of janky code and janky game play. if I culd go back I whuld defenstrate all the skripts about the enemy and restart.
most problems came from bad planing and a ever changing scope. the game works and the enemy funktions just very janky. I did lern a lot from this project which defenetly helped in later projects.

enouth whit my doom and gloom time to describe the hideus spagety that shuld be defenstrated.

in the original consept for the game there was supose to be 3 bosses. one for each programer and since I was supose to make the first one I tried to create a modular system that the others culd use for there bosses.
Do to that I tried to use inheritence to make it easy but it ended upp over complicating the scripts since some are 4 inherentes deep. the single some what good use was for the phase system.

I used a state machene to determine the phase the boss was on. For some reason I made it in the uppdate funktion instead of making a seperet funktion that only run when the boss takes damage.

<details>

<summary>State machine</summary>

  ```csharp
private void Awake()
    {
        phase = phases[activePhase];
        health = GetComponent<Enemi_health>();
        enemy = GetComponent<Base_enemy>();
    }

    public void Update()
    {
        if (health.currentHP < health.MaxHP * phase3HPPercent && phase == phases[1] && !PhaseSwitch)
        {
            activePhase = 2;
            StartCoroutine(wait(4, activePhase));
        }
        else if (health.currentHP < health.MaxHP * phase2HPPercent && phase == phases[0] && !PhaseSwitch)
        {
            activePhase = 1;
            StartCoroutine(wait(4, activePhase));
        }
        phase.phase();
    }

    IEnumerator wait(float time, int NewPhase)
    {
        enemy.stop();
        enemy.atta = false;
        PhaseSwitch = true;
        health.source.PlayOneShot(PhaseSwitchSound, 0.5f);
        yield return new WaitForSeconds(time);
        phase = phases[NewPhase];
        PhaseSwitch = false;
        enemy.atta = true;
        enemy.start();
    }
```
</details>

The phases themselves inherited from a script that held all the boss's attacks wich inherits from a BasePhase script.

It was made early on when we were still planing on have multiple bosses and for that purpose it whuld have worked fine since the BasePhase script culd hold common funktions for all bosses. While the attack onse culd hold there more specialised attack funktions. and the actual phase scripts culd hold the logic of when tings happens.

<details>

<summary>BasePhaseScript</summary>
  
```csharp
 public void resetpositon()
    {
        agent.SetDestination(transform.position);
    }
    public void Move()
    {
        agent.SetDestination(player.transform.position);
    }

```
  
</details>

<details>
<summary>EnemyAttacks</summary>
  
```csharp
  
private void Awake()
    {
        UrchinSpawnerScript = FindObjectOfType<UrchinSpawner>();
        enemy = GetComponentInParent<Base_enemy>();
        agent = GetComponentInParent<NavMeshAgent>();
        weekpoint = transform.parent.GetComponentInChildren<weekpoint>();
    }
    
 
    public IEnumerator cooldown(float t)
    {
        
        enemy.atta = true;
        attack.parent.start();
        attack.still = false;
        yield return null;
    }

    public IEnumerator dublewave(int amount)
    {
        float firstWaveDelay = 2.9f;
        float secondWaveDelay = 1.0f;
        stunable = true;

        if (amount == 1)
        {
            SoundcueHandler.PlayWaveCue();
            StartCoroutine(enemy.weakPoint.SingleShockwave());
            enemy.GetComponentInChildren<MantisAnimator>().anim.SetTrigger("Shockwave");
        }
        else if(amount > 1)
        {
            SoundcueHandler.PlayDoubleWaveCue();
            StartCoroutine(enemy.weakPoint.DoubleShockwave());
            amount = 2;
            enemy.GetComponentInChildren<MantisAnimator>().anim.SetTrigger("Double Shockwave");
            firstWaveDelay = 2.0f;
            secondWaveDelay = 0.85f;
        }

        WaveAnim = true;
        yield return new WaitForSeconds(0.7f);
        StartCoroutine(weekpoint.MoveCollider());
        yield return new WaitForSeconds(firstWaveDelay - 0.7f);
        enemy.stop();

        if (!enemy.volnereble) 
        {
            for (int j = 0; j < amount; j++)
            {
                if (!enemy.volnereble)
                {
                    WaveAnim = false;
                    attack.shockwave(shockwavespeed, shockwavezise, shockwaverange);
                    if (j == 0)
                    {
                       //spawning urchins
                       UrchinSpawnerScript.WhichPhaseForUrchin();
                    }
                    yield return new WaitForSeconds(secondWaveDelay);
                    WaveAnim = true;
                }
            }
        }

        WaveAnim = false;
        stunable = false;
        enemy.start();
        StartCoroutine(cooldown(shockwavespeed));
    }

    public IEnumerator elestickdelay(float time)
    {
        StartCoroutine(enemy.weakPoint.Fist());

        ElastickAnim = true;
        SoundcueHandler.PlayFistCue();
        enemy.atta = false;
        stunable = true;
        enemy.GetComponentInChildren<MantisAnimator>().anim.SetTrigger("Punch 0");
        yield return new WaitForSeconds(time);
        ElastickAnim = false;
        if (!enemy.volnereble && !PlayerSlice.SliceMode())
        {
            attack.Elastick(elastickrange, elastickspeed, elastickreturnspeed);
        }
        else
        {
            enemy.atta = true;
            stunable = false;
        }
    }
```
  
</details>

<details>

<summary>Phase 3</summary>

```csharp
 public override void phase()
    {
        if (enemy.Range(startpunchrange) && !enemy.volnereble && !PlayerSlice.SliceMode())
        {
            resetpositon();
        }
        if (enemy.Range(startelastickrange) && enemy.atta && !enemy.volnereble && !PlayerSlice.SliceMode())
        {
            enemy.atta = false;
            if (i % 4 == 0)
            {
                StartCoroutine(elestickdelay(elastickdelai));
            }
            else
            {
                attack.still = true;
                StopCoroutine(dublewave(wavemount));
                wavemount = Random.Range(1, 5);
                StartCoroutine(dublewave(wavemount));
            }
            i++;
            resetpositon();
        }
        else if (!enemy.Range(startpunchrange) && !enemy.volnereble && !PlayerSlice.SliceMode())
        {
            enemy.agent.SetDestination(enemy.target.position);
        }
        else
        {
            resetpositon();
        }
        
    }
```
  
</details>

One problem whit it is that every ting is not centralized to those scripts. Several important parts of the attacks are in a seperate script. Some are there becos we did not originaly plan on having multeple phases and whuld have ben a significant amount of work to find all references to it and change them. Others are there becose they were suposed to be used by all bosses.

I origanaly felt realy bad about this project and was thinking that it was all trash. But now after re reading my code I can see that I had a good base idea for how it whuld work whit multiple bosses.


## my other contributions

The main menue and the diferent windows there






