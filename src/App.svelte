<script lang="ts">
  async function getBranchName(): Promise<void> {
    let type = await getElementText('[id^="type-val"]');
    let key = await getElementText('[id^="key-val"]');
    let summary = await getElementText('[id^="summary-val"]');
    summary = sanatizeForGitBranch(summary);

    if (type.includes("Bug")) {
      let branch = `bugfix/${key}_${summary}`;
      writeToClipboard(branch);
      window.close();
      return;
    }
    if (type.includes("User Story")) {
      let branch = `feature/${key}_${summary}/develop`;
      writeToClipboard(branch);
      window.close();
      return;
    }
    if (type.includes("Technical Task")) {
      let parentDescription = await getElementText('[id^="parent_issue_summary"]');
      parentDescription = sanatizeForGitBranch(parentDescription);
      let branch = `feature/${parentDescription}/${key}_${summary}`;
      writeToClipboard(branch);
      window.close();
      return;
    }
    if (type.includes("Technical Debt")) {
      let branch = `technical-debt/${key}_${summary}`;
      writeToClipboard(branch);
      window.close();
      return;
    }
    let branch = `${key}_${summary}`;
    writeToClipboard(branch);
    window.close();
  }

	async function getElementText(query: string): Promise<string> {
      return await executeScript(`document.querySelector('${query}').textContent`);
  }

  async function executeScript(code: string): Promise<any> {
    const tabs = await browser.tabs.query({active: true, currentWindow: true});
    const tab = tabs[0];
    const results = await browser.tabs.executeScript(tab.id, {code: code});
    return results[0];
  }

	function writeToClipboard(text: string) {
		navigator.clipboard.writeText(text);
	}

	const germanSpecialChars = {
    '\u00dc': 'UE',
    '\u00c4': 'AE',
    '\u00d6': 'OE',
    '\u00fc': 'ue',
    '\u00e4': 'ae',
    '\u00f6': 'oe',
    '\u00df': 'ss',
  }

  function replaceGermanChars(str: string): string {
    return str
      .replace(/[\u00dc|\u00c4|\u00d6][a-z]/g, (a) => {
        const big = germanSpecialChars[a.slice(0, 1)];
        return big.charAt(0) + big.charAt(1).toLowerCase() + a.slice(1);
      })
      .replace(new RegExp('['+Object.keys(germanSpecialChars).join('|')+']',"g"),
        (a) => germanSpecialChars[a]
      );
  }

  function sanatizeForGitBranch(str: string): string {
    return replaceGermanChars(str)
      .replace("[FA]", "")
      .replace("[UX]", "")
      .replace("\"", "")
      .replace("+", "Plus")
      .replace(" ", "_")
      .replace(/[^a-zA-Z0-9\-]/g, '_')
      .replace("__", "_")
      .replace("__", "_")
      .replace(/^_/g, '');
  }
</script>

<main>
    <button on:click={() => getBranchName()}>BranchName</button>
</main>

<style lang="scss">
	main {
		padding: 1em;
		width: 100px;
		height: 40px;
        overflow: hidden;
	}
</style>