<script>
  import { Plus, Trash, ImagePlus, RotateCw, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' }
    },
    items: [{ desc: '', price: '', quantity: '' }],
    taxPercent: '',
    discountPercent: '',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: ''
    },
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar' },
    { code: 'EUR', symbol: '€', name: 'Euro' },
    { code: 'GBP', symbol: '£', name: 'British Pound' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar' },
    { code: 'CAD', symbol: 'C$', name: 'Canadian Dollar' },
    { code: 'CHF', symbol: 'CHF', name: 'Swiss Franc' },
    { code: 'CNY', symbol: '¥', name: 'Chinese Yuan' },
    { code: 'INR', symbol: '₹', name: 'Indian Rupee' },
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble' },
    { code: 'BRL', symbol: 'R$', name: 'Brazilian Real' },
    { code: 'ZAR', symbol: 'R', name: 'South African Rand' },
    { code: 'MXN', symbol: '$', name: 'Mexican Peso' },
    { code: 'NZD', symbol: 'NZ$', name: 'New Zealand Dollar' },
    { code: 'SGD', symbol: 'S$', name: 'Singapore Dollar' }
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  // Utils --------------------------------------------------------------

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (
      file &&
      (file.type === 'image/png' || file.type === 'image/jpeg' || file.type === 'image/webp')
    ) {
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      alert('Please select a valid image (.png, .jpg, or .webp)');
    }
  }

  function save() {
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    if (itemDesc != '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [...appState.items, { desc: itemDesc, price: itemPrice, quantity: itemQty }];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('descBox').focus();
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Wayne Enterprises', logo: '' },
      invoice: {
        number: '1277',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Enter Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Enter Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' }
      },
      items: [
        { desc: 'Batmobile Repair', price: '15000', quantity: '1' },
        { desc: 'Grappling Hooks', price: '500', quantity: '20' },
        { desc: 'Interdimensional Portal Gun', price: '200000', quantity: '1' }
      ],
      taxPercent: '18',
      discountPercent: '10',
      note: '',
      payment: {
        accountNumber: '9876 5432 1098',
        accountName: 'Wayne Enterprises',
        bank: 'Gotham National Bank',
        ifsc: 'GNB0001234',
        swiftCode: 'GNBKUS6S'
      },
      currency: 'USD'
    };

    save();
  }

  // Startup ------------------------------------------------------------

  onMount(() => {
    const savedData = localStorage.getItem('invoiceData');
    if (savedData) {
      appState = JSON.parse(savedData);
    } else {
      reset();
    }
  });

  let itemDesc = '';
  let itemPrice = 1;
  let itemQty = 1;

  $: subTotal = appState.items.reduce((total, item) => {
    return total + item.price * item.quantity;
  }, 0);

  $: discountAmount = subTotal * (+appState.discountPercent || 0) / 100;
  $: taxableAmount = subTotal - discountAmount;
  $: taxAmount = taxableAmount * (+appState.taxPercent || 0) / 100;
  $: totalDue = taxableAmount + taxAmount;

  // Toggles
  let isDiscountEnabled = true;
  let isTaxEnabled = true;
</script>

<svelte:head>
  <title>Billie - No Strings Attached Invoice Generator</title>
  <meta
    name="description"
    content="Create and download invoices instantly, with zero signups or tracking."
  />
</svelte:head>

<svelte:body on:click={() => save()} />

<div
  class="max-w-screen-md mx-auto px-6 py-8 flex flex-col space-y-6 font-montserrat bg-gradient-to-b from-[#D2E0FB] to-[#FEF9D9] border border-[#DEE5D4] rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none"
>
  <!-- Company & Invoice Details --------------------------------------->
  <div class="flex flex-row justify-between items-start">
    <div class="flex flex-col gap-4">
      <div class="print:hidden">
        {#if appState.company.logo}
          <div class="flex flex-row items-center space-x-4">
            <div class="flex flex-col border border-[#DEE5D4] rounded-xl">
              <button
                class="p-2 hover:bg-[#DEE5D4] rounded"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                <Trash />
              </button>
              <button class="p-2 hover:bg-[#DEE5D4] rounded" on:click={() => document.getElementById('imageInput').click()}>
                <ImagePlus />
                <input
                  id="imageInput"
                  type="file"
                  accept=".png,.jpg,.jpeg,.webp"
                  on:change={handleImageChange}
                  class="hidden"
                />
              </button>
            </div>
            <img
              src={appState.company.logo}
              alt="Company Logo"
              class="max-h-18 max-w-40 object-cover m-0"
            />
          </div>
        {:else}
          <button
            class="p-2 flex flex-row gap-2 rounded-lg border border-[#DEE5D4] cursor-pointer hover:bg-[#DEE5D4] print:hidden"
            on:click={() => document.getElementById('imageInput').click()}
          >
            <ImagePlus />
            <p class="text-sm text-[#333]">Click to select an image for logo</p>
            <input
              id="imageInput"
              type="file"
              accept=".png,.jpg,.jpeg,.webp"
              on:change={handleImageChange}
              class="hidden"
            />
          </button>
        {/if}
      </div>
      <input class="font-bold text-xl border border-[#DEE5D4] p-3 rounded-lg focus:outline-none focus:border-[#8EACCD]" type="text" bind:value={appState.company.name} />
    </div>

    <div class="relative flex flex-col items-end gap-2">
      <h2 class="font-bold text-2xl text-[#1E6F5C] text-right">
        INVOICE :
        <input
          type="text"
          size="4"
          placeholder="2341"
          maxlength="4"
          bind:value={appState.invoice.number}
          class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
        />
      </h2>
      <button
        on:click={() => reset()}
        class="p-2 rounded-lg hover:bg-[#DEE5D4] transition-all duration-100 ease-in-out print:hidden"
      >
        <RotateCw strokeWidth={1.5} />
      </button>
      <div class="flex flex-col gap-1 print:hidden">
        <p>
          Created :
          <input
            bind:value={appState.invoice.created}
            type="text"
            size="10"
            placeholder="Date Created"
            maxlength="13"
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
          />
        </p>
        <p>
          Due :
          <input
            bind:value={appState.invoice.due}
            type="text"
            size="10"
            placeholder="Due Date"
            maxlength="13"
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
          />
        </p>
      </div>
      <select bind:value={appState.currency} class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD] print:hidden">
        {#each currencyOptions as option}
          <option value={option.code}>{option.code} - {option.name}</option>
        {/each}
      </select>
    </div>
  </div>

  <hr class="border-[#DEE5D4] print:hidden" />

  <!-- From & To  ------------------------------------------------------>

  <div class="flex flex-row justify-between">
    <div>
      <h4 class="mb-2 font-bold">From</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Enter sender's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
      ></textarea>
      <div class="mt-2">
        <p>
          Tax Name:
          <input
            type="text"
            bind:value={appState.invoice.fromTaxIdName}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="Enter Tax Name"
          />
        </p>
        <p>
          Tax ID:
          <input
            type="text"
            bind:value={appState.invoice.fromTaxId}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="Enter Tax ID"
          />
        </p>
        <p>
          Email:
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="bruce@wayneenterprises.com"
          />
        </p>
        <p>
          Phone:
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="+1 555-0199"
          />
        </p>
      </div>
    </div>
    <div>
      <h4 class="mb-2 font-bold">To</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Enter recipient's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
      ></textarea>
      <div class="mt-2">
        <p>
          Tax Name:
          <input
            type="text"
            bind:value={appState.invoice.toTaxIdName}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="Enter Tax Name"
          />
        </p>
        <p>
          Tax ID:
          <input
            type="text"
            bind:value={appState.invoice.toTaxId}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="Enter Tax ID"
          />
        </p>
        <p>
          Email:
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="rick@c137.com"
          />
        </p>
        <p>
          Phone:
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
            placeholder="+1 555-0118"
          />
        </p>
      </div>
    </div>
  </div>

  <!-- New Item Form --------------------------------------------------->

  <form class="flex flex-row justify-between gap-2">
    <button
      disabled={itemDesc == ''}
      on:click={() => addItem()}
      class="p-2 rounded-full bg-[#DEE5D4] hover:bg-[#CFCFCF] transition-all duration-100 ease-in-out print:hidden"
    >
      <Plus stroke="#557571" />
    </button>

    <input
      id="descBox"
      type="text"
      bind:value={itemDesc}
      placeholder="Item description"
      class="border border-[#DEE5D4] rounded-lg p-3 focus:outline-none focus:border-[#8EACCD] grow"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemPrice}
      placeholder="Unit Price"
      min="1"
      class="border border-[#DEE5D4] rounded-lg p-3 focus:outline-none focus:border-[#8EACCD] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemQty}
      placeholder="Quantity"
      min="1"
      step="1"
      class="border border-[#DEE5D4] rounded-lg p-3 focus:outline-none focus:border-[#8EACCD] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <p class="border border-[#DEE5D4] p-3 w-32 text-right">
      {getCurrencySymbol(appState.currency)}{itemPrice * itemQty}
    </p>
  </form>

  <!-- Table ----------------------------------------------------------->

  <div class="flex flex-col mt-4">
    <div class="flex flex-row bg-[#FAFAFA] border border-[#DEE5D4]">
      <p class="font-bold p-3 border-r border-[#DEE5D4] grow">Item Description</p>
      <p class="font-bold p-3 border-r border-[#DEE5D4] w-32">Unit Price</p>
      <p class="font-bold p-3 border-r border-[#DEE5D4] w-24">Qty</p>
      <p class="font-bold p-3 w-32 text-right">Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-row even:bg-[#F8F8F8] border border-[#DEE5D4]">
        <button
          on:click={() => deleteItem(index)}
          class="p-3 hover:bg-[#DEE5D4] transition-all duration-100 ease-in-out rounded print:hidden"
        >
          <Trash color="#C96868" />
        </button>

        <p contenteditable="true" class="p-3 border-r border-[#DEE5D4] grow">{item.desc}</p>
        <input class="p-3 border-r border-[#DEE5D4] w-32" type="text" bind:value={item.price} />
        <input class="p-3 border-r border-[#DEE5D4] w-24" type="text" bind:value={item.quantity} />
        <p class="p-3 w-32 text-right">{getCurrencySymbol(appState.currency)}{item.price * item.quantity}</p>
      </div>
    {/each}

    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#DEE5D4] items-center">
      <button
        class="toggle-switch ml-2 mr-3"
        on:click={() => (isDiscountEnabled = !isDiscountEnabled)}
      ></button>
      <p class="p-3 grow text-right font-bold">Discount %</p>
      <input
        class="p-3 w-32 text-right"
        type="text"
        bind:value={appState.discountPercent}
        disabled={!isDiscountEnabled}
      />
    </div>
    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#DEE5D4] items-center">
      <button class="toggle-switch ml-2 mr-3" on:click={() => (isTaxEnabled = !isTaxEnabled)}></button>
      <p class="p-3 grow text-right font-bold">Tax %</p>
      <input class="p-3 w-32 text-right" type="text" bind:value={appState.taxPercent} disabled={!isTaxEnabled} />
    </div>
    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#DEE5D4]">
      <p class="p-3 grow text-right font-bold">Total Due</p>
      <p class="p-3 w-32 text-right">{getCurrencySymbol(appState.currency)}{totalDue}</p>
    </div>
  </div>

  <!-- Payment Info Section Redesigned --------------------------------->
  <div class="mt-4">
    <h4 class="mb-2 font-bold">Payment Info</h4>
    <div class="grid grid-cols-2 gap-4">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account</label>
        <input
          type="text"
          maxlength="20"
          placeholder="987654321098"
          bind:value={appState.payment.accountNumber}
          class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">A/C Name</label>
        <input
          type="text"
          maxlength="28"
          placeholder="Wayne Enterprises"
          bind:value={appState.payment.accountName}
          class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Bank Name</label>
        <input
          type="text"
          maxlength="28"
          placeholder="Gotham National Bank"
          bind:value={appState.payment.bank}
          class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">IFSC</label>
        <input
          type="text"
          maxlength="11"
          placeholder="GNB0001234"
          bind:value={appState.payment.ifsc}
          class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">SWIFT Code</label>
        <input
          type="text"
          maxlength="11"
          placeholder="GNBKUS6S"
          bind:value={appState.payment.swiftCode}
          class="border border-[#DEE5D4] p-2 rounded-lg focus:outline-none focus:border-[#8EACCD]"
        />
      </div>
    </div>
  </div>

  <!-- Download Button -------------------------------------------------->

  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-3 rounded-lg bg-[#DEE5D4] text-[#333] font-semibold flex items-center gap-2 hover:bg-[#CFCFCF] transition-transform duration-150"
    >
      <Download class="w-5 h-5" />
      <span>Download Invoice</span>
    </button>
  </div>
</div>

<style>
  .toggle-switch {
    width: 40px;
    height: 20px;
    background-color: #dee5d4;
    border-radius: 20px;
    position: relative;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .toggle-switch:before {
    content: '';
    position: absolute;
    top: 2px;
    left: 2px;
    width: 16px;
    height: 16px;
    background-color: #8eaccd;
    border-radius: 50%;
    transition: all 0.3s;
  }

  .toggle-switch.active {
    background-color: #8eaccd;
  }

  .toggle-switch.active:before {
    transform: translateX(20px);
    background-color: #feffd9;
  }
</style>
