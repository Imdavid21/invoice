import React, { useState, useEffect } from 'react';
import { GitHub, Linkedin, Coffee } from 'lucide-react';

const FunFooter = ({ repo = 'repo-name' }) => {
  const [year, setYear] = useState(new Date().getFullYear());
  const [hover, setHover] = useState('');

  useEffect(() => {
    const timer = setInterval(() => {
      setYear(new Date().getFullYear());
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  const iconStyle = "w-6 h-6 inline-block transition-transform duration-300 ease-in-out";
  const linkStyle = "font-semibold underline underline-offset-4 decoration-2 transition-colors duration-300";

  return (
    <footer className="print:hidden text-stone-800 my-16 p-4 bg-gradient-to-r from-blue-100 to-purple-100 rounded-lg shadow-md">
      <div className="text-center space-y-4">
        <p className="text-xl font-bold">
          © {year} - Time flies when you're coding!
        </p>
        <div className="flex justify-center items-center space-x-4">
          <a
            href={`https://github.com/commitsovercoffee/${repo}`}
            target="_blank"
            rel="noopener noreferrer"
            className={`${linkStyle} hover:text-blue-600`}
            onMouseEnter={() => setHover('github')}
            onMouseLeave={() => setHover('')}
          >
            <GitHub className={`${iconStyle} ${hover === 'github' ? 'rotate-12' : ''}`} />
            <span className="ml-2">{repo}</span>
          </a>
          <span className="text-2xl">·</span>
          <a
            href="https://www.linkedin.com/in/uddalak-das-crypto-nft-defi/"
            target="_blank"
            rel="noopener noreferrer"
            className={`${linkStyle} hover:text-indigo-600`}
            onMouseEnter={() => setHover('linkedin')}
            onMouseLeave={() => setHover('')}
          >
            <Linkedin className={`${iconStyle} ${hover === 'linkedin' ? 'rotate-12' : ''}`} />
            <span className="ml-2">Uddalak Das</span>
          </a>
        </div>
        <p className="italic text-sm">
          Built with
          <Coffee className={`${iconStyle} mx-1 text-brown-600 ${hover === 'coffee' ? 'animate-bounce' : ''}`} 
                  onMouseEnter={() => setHover('coffee')} 
                  onMouseLeave={() => setHover('')} />
          and a lot of debugging!
        </p>
      </div>
    </footer>
  );
};

export default FunFooter;
